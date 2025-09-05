# 【0円開発】 Docker + Ollama でローカルLLMを使ったAI機能実装

## はじめに
個人開発をしていると、ちょっとした機能追加でも「本番で動かすならAPIを使うしかないけど、開発中にずっと叩いてたらお金がかかるな…」というジレンマに陥りがちです。特にAIチャットのようにリクエストが多くなる機能は、開発段階で試行錯誤するたびに課金されてしまい、学生や個人開発者にとっては大きな負担になりかねません。そこで私は、「開発時は完全無料で動かしたい、でも本番は信頼できる商用サービスを使いたい」という2つの条件を両立させるために、開発環境＝Docker + Ollama、本番環境＝Gemini API という構成をとることにしました。

本記事では、この構成をどのように導入したのか、また実際に使ってみてのメリットや工夫した点を紹介していきます。

## 技術スタック

### バックエンド

* **Laravel 12**：APIサーバー
* **MySQL**：メタデータ保存（会話内容は保存しない）
* **Docker + Laravel Sail**：開発環境構築
* **AIプロバイダー**

  * 開発環境：**Ollama (`mistral:latest`)**
  * 本番環境：**Gemini API (`gemini-pro`)**

### フロントエンド

* **SwiftUI**

### インフラ

* **開発環境**：Docker Compose（Ollamaコンテナを追加）
* **本番環境**：Render（APIホスティング）、Gemini API（AI処理）


## 課題と解決策

### 開発時の課題

個人開発でAI機能を実装しようとすると、**開発中に発生するAPI利用料**が意外と無視できません。
テストのたびにリクエストを投げるので、ちょっと動作確認しただけで課金が積み重なってしまいます。

### 本番の要件

一方で、リリース後の本番環境では以下が必須になります：

* **安定したレスポンス**（ユーザー体験を壊さない）
* **高可用性**（サービスが止まらない）
* **スケーラビリティ**（ユーザー数が増えても対応できる）

つまり、「開発」と「本番」では求められる条件が大きく異なるわけです。

### 解決策

そこで採用したのが、**環境ごとにAIプロバイダーを切り替える仕組み**です。

* **開発環境** → Docker上でOllamaを動かし、ローカルでLLMを利用（コスト0円 & ネットワーク非依存）
* **本番環境** → Google Gemini APIを利用し、クラウドの安定性とスケーラビリティを確保

切り替えは **環境変数1つ** で完了。コードの変更は不要で、環境だけを切り替えるシンプルなアーキテクチャにしました。

## 開発環境：Docker + Ollama

### Ollamaを選んだ理由

開発環境におけるAIプロバイダーとして **Ollama** を選択したのには、いくつかの明確な理由があります。特に個人開発では「コストを抑えつつ、実際の利用シナリオに近い形で試したい」というニーズが強く、その点でOllamaは非常に相性が良いと感じました。

* **完全無料でローカル実行可能**：一度モデルをダウンロードすれば、追加コストなしで使い続けられます。
* **Docker対応で構築が簡単**：`docker compose up -d` だけで環境を立ち上げられ、Laravel Sailとの統合も容易です。
* **多様なモデルをサポート**：今回は `mistral:latest` を利用しましたが、他のオープンモデルにも柔軟に対応できます。
* **API互換でコード変更不要**：外部APIと似た形式で呼び出せるため、本番用プロバイダーとの切り替えが容易です。
* **オフライン利用可能**：初回のモデル取得以降は、ネットワークに依存せず、ローカルの `localhost` へのアクセスだけで動作します。

これらの特徴により、Ollamaは「開発中は完全無料で安全に試せて、本番移行もスムーズにできる」理想的な選択肢となりました。

### Docker設定

Laravel SailにOllama追加：

<details><summary>docker-compose.yml</summary>

```yaml
services:
    # 既存のLaravelサービス...
    
    ollama:
        image: 'ollama/ollama:latest'
        ports:
            - '${FORWARD_OLLAMA_PORT:-11434}:11434'
        volumes:
            - 'sail-ollama:/root/.ollama'
        networks:
            - sail
        environment:
            - OLLAMA_HOST=0.0.0.0
            - OLLAMA_ORIGINS=*
        healthcheck:
            test:
                - CMD
                - curl
                - '-f'
                - 'http://localhost:11434/api/tags'
            retries: 3
            timeout: 10s
            interval: 30s
        restart: unless-stopped

    ollama-init:
        image: 'ollama/ollama:latest'
        volumes:
            - 'sail-ollama:/root/.ollama'
            - './docker/ollama/init-ollama.sh:/init-ollama.sh'
        networks:
            - sail
        environment:
            - OLLAMA_HOST=ollama:11434
            - OLLAMA_MODEL=${OLLAMA_MODEL:-mistral:latest}
        depends_on:
            ollama:
                condition: service_healthy
        command: ["/bin/bash", "/init-ollama.sh"]
        restart: "no"

volumes:
    sail-ollama:
        driver: local
```
</details>







### 自動セットアップ

モデルダウンロードを自動化：

<details><summary>init-ollama.sh</summary>

```bash
#!/bin/bash
# docker/ollama/init-ollama.sh

set -e

echo "Starting Ollama initialization..."

export OLLAMA_HOST=${OLLAMA_HOST:-http://ollama:11434}

# Ollamaサービスの起動を待機
echo "Waiting for Ollama service to start at $OLLAMA_HOST..."
until curl -f "$OLLAMA_HOST/api/tags" >/dev/null 2>&1; do
    echo "Waiting for Ollama to be ready..."
    sleep 5
done

echo "Ollama service is ready!"

MODEL_NAME=${OLLAMA_MODEL:-llama2}

# モデルが既にダウンロード済みかチェック
if ollama list | grep -q "$MODEL_NAME" 2>/dev/null; then
    echo "Model '$MODEL_NAME' is already available."
else
    echo "Downloading model '$MODEL_NAME'... This may take a while."
    ollama pull "$MODEL_NAME"
    echo "Model '$MODEL_NAME' downloaded successfully!"
fi

# モデルのテスト
echo "Testing model with a simple prompt..."
echo "Hello, I am Kumamon AI!" | ollama run "$MODEL_NAME" || echo "Model test completed"

echo "Ollama initialization completed successfully!"
```

</details>














## Laravel：環境切り替えの実装

### Strategy パターン

環境変数でプロバイダー切り替え：

<details><summary>AIProviderInterface</summary>

```php
<?php

namespace App\Services\AI;

interface AIProviderInterface
{
    public function generateResponse(string $message): string;
    public function isAvailable(): bool;
}
```

</details>




<details><summary>OllamaProvider（開発環境用）</summary>

```php
<?php

namespace App\Services\AI\Providers;

use App\Services\AI\AIProviderInterface;
use Illuminate\Support\Facades\Http;

class OllamaProvider implements AIProviderInterface
{
    private string $url;
    private string $model;
    private int $timeout;

    public function __construct()
    {
        $this->url = config('ai.ollama.url');
        $this->model = config('ai.ollama.model');
        $this->timeout = config('ai.ollama.timeout');
    }

    public function generateResponse(string $message): string
    {
        $response = Http::timeout($this->timeout)
            ->post("{$this->url}/api/generate", [
                'model' => $this->model,
                'prompt' => $message,
                'stream' => false,
            ]);

        if (!$response->successful()) {
            throw new Exception("Ollama API error: " . $response->body());
        }

        $data = $response->json();
        
        if (!isset($data['response'])) {
            throw new Exception("Invalid response format from Ollama");
        }

        return $data['response'];
    }

    public function isAvailable(): bool
    {
        try {
            $response = Http::timeout(5)->get("{$this->url}/api/tags");
            return $response->successful();
        } catch (Exception $e) {
            return false;
        }
    }
}
```

</details>



<details><summary>GeminiProvider（本番環境用）</summary>


```php
<?php

namespace App\Services\AI\Providers;

use App\Services\AI\AIProviderInterface;
use Illuminate\Support\Facades\Http;

class GeminiProvider implements AIProviderInterface
{
    private string $apiKey;
    private string $model;
    private string $apiUrl;
    private int $timeout;

    public function __construct()
    {
        $this->apiKey = config('ai.gemini.api_key');
        $this->model = config('ai.gemini.model');
        $this->apiUrl = config('ai.gemini.api_url');
        $this->timeout = config('ai.gemini.timeout');
    }

    public function generateResponse(string $message): string
    {
        $url = "{$this->apiUrl}/models/{$this->model}:generateContent?key={$this->apiKey}";
        
        $response = Http::timeout($this->timeout)
            ->withHeaders(['Content-Type' => 'application/json'])
            ->post($url, [
                'contents' => [
                    [
                        'parts' => [
                            ['text' => $message]
                        ]
                    ]
                ],
                'generationConfig' => [
                    'temperature' => 0.7,
                    'topK' => 40,
                    'topP' => 0.95,
                    'maxOutputTokens' => 1024,
                ]
            ]);

        if (!$response->successful()) {
            $errorBody = $response->json();
            $errorMessage = $errorBody['error']['message'] ?? $response->body();
            throw new Exception("Gemini API error: " . $errorMessage);
        }

        $data = $response->json();
        return $data['candidates'][0]['content']['parts'][0]['text'];
    }

    public function isAvailable(): bool
    {
        // Gemini APIの可用性チェック実装
        // ...
    }
}
```

</details>



### AIService

プロバイダー選択とメタデータ記録：


<details><summary>AIService</summary>

```php
<?php

namespace App\Services\AI;

use App\Models\AIChatLog;
use App\Services\AI\Providers\OllamaProvider;
use App\Services\AI\Providers\GeminiProvider;

class AIService
{
    public function chat(string $message, int $userId): array
    {
        $startTime = microtime(true);
        $provider = $this->getProvider();
        $providerName = $this->getProviderName();
        
        // チャットログの作成
        $chatLog = AIChatLog::create([
            'user_id' => $userId,
            'provider' => $providerName,
            'request_timestamp' => now(),
        ]);

        try {
            // AI応答の生成
            $response = $provider->generateResponse($message);
            
            // レスポンス時間の計算
            $responseTime = (microtime(true) - $startTime) * 1000;
            
            // ログの更新
            $chatLog->update([
                'response_timestamp' => now(),
                'response_time_ms' => round($responseTime),
            ]);

            return [
                'message' => $response,
                'timestamp' => now()->toISOString(),
                'provider' => $providerName,
                'response_time_ms' => round($responseTime),
            ];
        } catch (Exception $e) {
            Log::error('AI Service Error', [
                'provider' => $providerName,
                'user_id' => $userId,
                'error' => $e->getMessage(),
            ]);
            
            throw $e;
        }
    }

    private function getProvider(): AIProviderInterface
    {
        $providerType = config('ai.provider', 'ollama');
        
        switch ($providerType) {
            case 'ollama':
                $provider = new OllamaProvider();
                break;
            case 'gemini':
                $provider = new GeminiProvider();
                break;
            default:
                throw new Exception("Unknown AI provider: {$providerType}");
        }

        if (!$provider->isAvailable()) {
            throw new Exception("AI provider '{$providerType}' is not available");
        }

        return $provider;
    }
}
```
</details>


### 設定ファイル

環境変数で切り替え：


<details><summary>comnfig</summary>

```php
<?php
// config/ai.php

return [
    'provider' => env('AI_PROVIDER', 'ollama'),

    'ollama' => [
        'url' => env('OLLAMA_URL', 'http://ollama:11434'),
        'model' => env('OLLAMA_MODEL', 'mistral:latest'),
        'timeout' => env('OLLAMA_TIMEOUT', 30),
    ],

    'gemini' => [
        'api_key' => env('GEMINI_API_KEY'),
        'model' => env('GEMINI_MODEL', 'gemini-pro'),
        'api_url' => env('GEMINI_API_URL', 'https://generativelanguage.googleapis.com/v1beta'),
        'timeout' => env('GEMINI_TIMEOUT', 30),
    ],

    'max_message_length' => env('AI_MAX_MESSAGE_LENGTH', 1000),
    'rate_limit_per_minute' => env('AI_RATE_LIMIT_PER_MINUTE', 10),
];
```

</details>

## iOS：API通信

### ChatAIService

統一インターフェース：

<details><summary>ChatAIService</summary>
```swift
class ChatAIService {
    static let shared = KumamonAIService()
    
    private let baseURL = ProcessInfo.processInfo.environment["API_BASE_URL"] ?? "https://localhost/api"
    private let timeout: TimeInterval = 30.0
    
    func sendMessage(_ message: String) async throws -> AIResponse {
        guard !message.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty else {
            throw ChatAIError.emptyMessage
        }
        
        let endpoint = "\(baseURL)/ai/chat"
        guard let url = URL(string: endpoint) else {
            throw ChatAIError.invalidURL
        }
        
        var request = URLRequest(url: url)
        request.httpMethod = "POST"
        request.setValue("application/json", forHTTPHeaderField: "Content-Type")
        request.timeoutInterval = timeout
        
        // 認証トークンの設定
        let token = getAuthToken()
        if !token.isEmpty {
            request.setValue("Bearer \(token)", forHTTPHeaderField: "Authorization")
        }
        
        // リクエストボディの作成
        let body = ["message": message]
        request.httpBody = try JSONSerialization.data(withJSONObject: body)
        
        let (data, response) = try await APISession.shared.session.data(for: request)
        guard let httpResponse = response as? HTTPURLResponse else {
            throw KumamonAIError.invalidResponse
        }
        
        return try handleResponse(data: data, statusCode: httpResponse.statusCode)
    }
    
    func checkServiceAvailability() async -> Bool {
        let endpoint = "\(baseURL)/ai/health"
        guard let url = URL(string: endpoint) else { return false }
        
        var request = URLRequest(url: url)
        request.httpMethod = "GET"
        request.timeoutInterval = 10.0
        
        do {
            let (_, response) = try await APISession.shared.session.data(for: request)
            guard let httpResponse = response as? HTTPURLResponse else { return false }
            return httpResponse.statusCode == 200
        } catch {
            return false
        }
    }
}
```

</details>








<details><summary>ChatAIViewModel（状態管理）</summary>
```swift
@MainActor
final class KumamonAIViewModel: ObservableObject {
    @Published var messages: [ChatMessage] = []
    @Published var isLoading: Bool = false
    @Published var errorMessage: String?
    @Published var inputText: String = ""
    @Published var isTyping: Bool = false
    @Published var isServiceAvailable: Bool = true
    
    private let aiService = ChatAIService.shared
    
    var canSendMessage: Bool {
        !inputText.trimmingCharacters(in: .whitespacesAndNewlines).isEmpty && 
        !isLoading && 
        !isTyping &&
        isServiceAvailable
    }
    
    func sendMessage(_ message: String? = nil) async {
        let messageToSend = message ?? inputText
        let sanitizedMessage = aiService.sanitizeMessage(messageToSend)
        
        guard aiService.isValidMessage(sanitizedMessage) else {
            errorMessage = "有効なメッセージを入力してください"
            return
        }
        
        // ユーザーメッセージを追加
        let userMessage = ChatMessage.userMessage(sanitizedMessage)
        messages.append(userMessage)
        
        inputText = ""
        isLoading = true
        isTyping = true
        
        do {
            let response = try await aiService.sendMessage(sanitizedMessage)
            let aiMessage = ChatMessage.aiMessage(response.message)
            messages.append(aiMessage)
        } catch let error as ChatAIError {
            await handleChatAIError(error)
        } catch {
            await handleGenericError(error)
        }
        
        isLoading = false
        isTyping = false
    }
    
    func clearChat() {
        messages.removeAll()
        inputText = ""
        errorMessage = nil
    }
}
```

</details>






<details><summary>ChatAIView（UI）</summary>

```swift
struct ChatAIView: View {
    @StateObject private var viewModel = ChatAIViewModel()
    @FocusState private var isInputFocused: Bool
    
    var body: some View {
        NavigationView {
            VStack(spacing: 0) {
                chatMessagesView
                messageInputView
            }
            .navigationTitle("AI")
            .toolbar {
                ToolbarItem(placement: .navigationBarTrailing) {
                    Button("クリア") {
                        viewModel.clearChat()
                    }
                    .disabled(viewModel.messages.isEmpty)
                }
            }
        }
        .onAppear {
            viewModel.checkServiceAvailability()
        }
    }
    
    private var chatMessagesView: some View {
        ScrollViewReader { proxy in
            ScrollView {
                LazyVStack(spacing: 12) {
                    if viewModel.messages.isEmpty {
                        emptyStateView
                    } else {
                        ForEach(viewModel.messages) { message in
                            MessageBubbleView(message: message)
                                .id(message.id)
                        }
                    }
                    
                    if viewModel.isTyping {
                        TypingIndicatorView()
                            .id("typing")
                    }
                }
                .padding(.horizontal, 16)
            }
            .onChange(of: viewModel.messages.count) { _ in
                withAnimation(.easeOut(duration: 0.3)) {
                    proxy.scrollTo("typing", anchor: .bottom)
                }
            }
        }
    }
    
    private var messageInputView: some View {
        HStack(alignment: .bottom, spacing: 12) {
            TextField("メッセージを入力...", text: $viewModel.inputText, axis: .vertical)
                .textFieldStyle(RoundedBorderTextFieldStyle())
                .focused($isInputFocused)
                .lineLimit(1...6)
            
            Button(action: sendMessage) {
                Image(systemName: "arrow.up.circle.fill")
                    .font(.title2)
                    .foregroundColor(viewModel.canSendMessage ? .blue : .gray)
            }
            .disabled(!viewModel.canSendMessage)
        }
        .padding(.horizontal, 16)
        .padding(.vertical, 12)
    }
    
    private func sendMessage() {
        isInputFocused = false
        Task {
            await viewModel.sendMessage()
        }
    }
}
```

</details>





## プライバシー配慮のデータベース設計

この仕組みでは、ユーザーが送受信した**メッセージの内容そのものは保存しません**。代わりに、リクエストやレスポンスのタイムスタンプ、レスポンスタイム、利用したプロバイダーなどの**メタデータのみを記録**する設計にしました。こうすることで、ユーザーの会話内容がデータベースに残らず、プライバシーが確保されます。また、個人情報漏洩のリスクを最小化でき、法規制の観点からも安全性が高まります。さらに、保存データが必要最小限になるため、データベースの効率性やパフォーマンスにも貢献します。

### AIChatLogモデル

```php
<?php

namespace App\Models;

use Illuminate\Database\Eloquent\Model;

class AIChatLog extends Model
{
    protected $fillable = [
        'user_id',
        'provider',
        'request_timestamp',
        'response_timestamp',
        'response_time_ms',
    ];

    protected $casts = [
        'request_timestamp' => 'datetime',
        'response_timestamp' => 'datetime',
        'response_time_ms' => 'integer',
    ];

    public function user(): BelongsTo
    {
        return $this->belongsTo(User::class);
    }
}
```


## 環境切り替えの実装

### 開発環境（.env）

```env
# Docker + Ollama（コスト0円）
AI_PROVIDER=ollama
OLLAMA_URL=http://ollama:11434
OLLAMA_MODEL=mistral:latest
OLLAMA_TIMEOUT=30
```

### 本番環境（.env）

```env
# Render + Gemini API（従量課金）
AI_PROVIDER=gemini
GEMINI_API_KEY=your_gemini_api_key
GEMINI_MODEL=gemini-pro
GEMINI_API_URL=https://generativelanguage.googleapis.com/v1beta
GEMINI_TIMEOUT=30
```

### 切り替え方法

環境変数1つで切り替え：

```bash
# 開発環境
export AI_PROVIDER=ollama

# 本番環境
export AI_PROVIDER=gemini
```



## 実際にやってみた感想

実際にDocker + Ollamaで開発環境を構築してAI機能を試してみると、いくつか気づきがありました。

### CPU使用率が限界突破することがある

コンテナの稼働中に `1000% / 800% (8 CPUs available)` のような状態になり、PCのリソースを食い尽くすことがありました。ローカル実行ならではの「マシン依存」の課題です。

### レスポンスは遅め

ネットワーク遅延がないのは嬉しいのですが、推論処理そのものが重いためクラウドAPIほどサクサクは動きません。

### 本番とのモデル差がある

開発ではOllamaの `mistral:latest` を使い、本番ではGemini APIを使うので、最終的にはプロンプト調整や必要に応じたチューニングを行う必要があります。

### それでも得られるメリット

とはいえ、UIのテストや開発サイクルの高速化という点では大いに効果を発揮しました。課金を気にせず無限に叩けるので、「AIが返答してくるフロー」の実装確認には最適です。

<img src="https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/4094814/ab8189be-6d8a-4d46-9816-13d98edec1d6.png" width="300">


## まとめ

今回、開発環境では Docker + Ollama を、本番環境では Gemini API を使い分ける仕組みを導入しました。環境変数ひとつで切り替えられるようにしたことで、開発中はコストをかけずに好きなだけ試せて、本番では安定したサービスを利用できる柔軟な構成になりました。
ローカル実行ならではのCPU負荷やレスポンス速度の課題はありますが、UIテストや開発サイクルの高速化には十分役立ちました。個人開発やプロトタイプの段階では特に有効なアプローチだと感じています。