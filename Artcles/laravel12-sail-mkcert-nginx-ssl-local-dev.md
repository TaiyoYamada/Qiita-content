ローカル開発でLaravelをHTTPS化（SSL化）したくて、**mkcert + Nginx** という構成で試してみました。**なぜこの構成を選んだのか**という比較や、  
実際の設定手順、やってみて気づいたことをまとめます。本番環境ではなく、あくまで**Docker + Laravel Sailを使ったローカル開発用のSSL化**です。

まだまだ改善の余地がある手探り状態ですが、同じように試してみたい方の参考になれば嬉しいです。

## 1. はじめに

開発中でも HTTPS 化をしておくと、以下のようなメリットがあります。

- 本番環境に近い動作確認ができる
- PWAや一部の API 連携で HTTPS が必須
- ブラウザのセキュリティ機能（Service Worker など）を有効化できる

今回は **Laravel 12 + Sail** 環境に **Nginx** を追加し、  
**mkcert** を使ってブラウザ警告なしのローカル証明書を発行しました。


## 2. 選択肢の比較と理由

### 2.1 Webサーバー比較（Apache vs Nginx）

| 項目 | Nginx | Apache |
|------|-------|--------|
| **パフォーマンス** | イベント駆動で高速 | プロセス/スレッド駆動でやや重い |
| **メモリ使用量** | 軽量 | 比較的多い |
| **設定の柔軟性** | シンプルな構成 | モジュール多めで高機能 |
| **静的コンテンツ配信** | 高速 | 相対的に遅い |
| **SSL終端処理** | 高性能 | 標準的 |
| **本番採用事例** | 多い（特に高トラフィック環境） | 多い |

今回は以下の理由で **Nginx** を選びました。

- Sailに簡単に追加できる
- 高速＆軽量で本番環境でもよく使われる
- リバースプロキシ構成との相性が良い

### 2.2 証明書比較（mkcert vs 自己署名）

| 項目 | mkcert | 自己署名 |
|------|--------|----------|
| **ブラウザ警告** | なし（ローカルCAを信頼） | あり（毎回警告） |
| **開発効率** | 高い | 低い |
| **セットアップ** | 初回のみ | プロジェクトごとに必要 |
| **チーム開発** | 各自がセットアップ | 証明書共有が必要 |
| **信頼性** | ローカルCAで安全 | ブラウザは信頼しない |

今回は以下の理由で **mkcert** を選びました。

- 複数プロジェクトでも使い回せる
- ブラウザ警告が出ない
- ローカル環境でも安全性を担保できる

---

## 3. 環境

- **OS**: macOS  
- **Laravel**: 12.x  
- **PHP**: 8.4 (Sail デフォルト)  
- **Docker**: 24.x  
- **Nginx**: 1.29.0 (Alpine)  
- **mkcert**: 最新版（brew インストール）


## 4. 手順

### 4.1 mkcert インストール

```bash
brew install mkcert
mkcert -install
```

### 4.2 証明書作成

```bash
mkdir -p docker/nginx/ssl
mkcert -key-file docker/nginx/ssl/server.key \
       -cert-file docker/nginx/ssl/server.crt \
       localhost 127.0.0.1 ::1
```

### 4.3 Nginx設定
`docker/nginx/nginx.conf`を作成。

```nginx
user nginx;
worker_processes auto;
error_log /var/log/nginx/error.log notice;
pid /var/run/nginx.pid;

events {
    worker_connections 1024;
    use epoll;
    multi_accept on;
}

http {
    include /etc/nginx/mime.types;
    default_type application/octet-stream;

    # ログ設定
    log_format main '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for"';

    access_log /var/log/nginx/access.log main;

    # パフォーマンス最適化
    sendfile on;
    tcp_nopush on;
    tcp_nodelay on;
    keepalive_timeout 65;
    types_hash_max_size 2048;

    # gzip圧縮
    gzip on;
    gzip_vary on;
    gzip_min_length 1024;
    gzip_types text/plain text/css text/xml text/javascript application/javascript application/xml+rss application/json;

    # HTTPからHTTPSへのリダイレクト
    server {
        listen 80;
        server_name localhost;
        
        return 301 https://$host$request_uri;
    }

    # HTTPS設定
    server {
        listen 443 ssl;
        http2 on;
        http3 on;
        add_header Alt-Svc 'h3=":443"; ma=86400' always;
        server_name localhost;

        ssl_certificate     /etc/nginx/ssl/server.crt;
        ssl_certificate_key /etc/nginx/ssl/server.key;

        # 開発ではコメントアウト推奨（戻せなくなるの防止）
        # add_header Strict-Transport-Security "max-age=31536000; includeSubDomains" always;

        client_max_body_size 50m;

        location / {
            proxy_pass http://laravel.test:80;        # ← :80 を明示
            proxy_http_version 1.1;
            proxy_set_header Host              $host;
            proxy_set_header X-Real-IP         $remote_addr;
            proxy_set_header X-Forwarded-For   $proxy_add_x_forwarded_for;
            proxy_set_header X-Forwarded-Proto https; # ← 明示
            proxy_set_header Upgrade           $http_upgrade;  # WS/HMR
            proxy_set_header Connection        "upgrade";      # WS/HMR
        }
    }
}
```

### 4.4 docker-compose.yml更新

```yaml
nginx:
    image: nginx:alpine
    ports:
        - "443:443"
    volumes:
        - .:/var/www/html:ro
        - ./docker/nginx/nginx.conf:/etc/nginx/nginx.conf:ro
        - ./docker/nginx/ssl:/etc/nginx/ssl:ro
    depends_on:
        - laravel.test
```
### 4.5 Laravel側設定
`app/Providers/AppServiceProvider.php`

```php
use Illuminate\Support\Facades\URL;

public function boot(): void
{
    if (request()->header('X-Forwarded-Proto') === 'https') {
        URL::forceScheme('https');
    }
}
```
## 5. 動作確認

```bash
# Nginx 設定テスト
docker compose exec nginx nginx -t


# HTTPS アクセス確認
curl -Ik https://localhost
```
ブラウザで`https://localhost`にアクセスし、確認。


## まとめ
今回、Docker + Laravel Sail の環境を mkcert と Nginx で SSL 化してみました。最初は「ローカルだしHTTPでいいかな」と思っていたんですが、PWAの検証や外部API連携を考えると、やっぱりHTTPS環境は必須だなと実感しました。

## 参考サイト

- [Laravel 公式ドキュメント（Sail）](https://laravel.com/docs/12.x/sail)  
- [Laravel 公式ドキュメント（HTTPS設定）](https://laravel.com/docs/12.x/requests#configuring-trusted-proxies)  
- [mkcert 公式GitHub](https://github.com/FiloSottile/mkcert)  
- [Nginx 公式ドキュメント](https://nginx.org/en/docs/)  
- [Mozilla SSL Configuration Generator](https://ssl-config.mozilla.org/)  
- [HTTP/3 + QUIC 対応設定（Nginx公式ブログ）](https://www.nginx.com/blog/introducing-technology-preview-nginx-support-for-quic-http-3/)  
