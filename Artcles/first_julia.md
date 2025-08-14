## 初めてのJulia体験記 – インストールから基本文法まで

## 1.はじめに

みなさん、Juliaをご存知でしょうか？
Juliaは「Cのように高速に動き、Pythonのように読みやすい」と評される、科学計算や機械学習の分野で注目されている言語です。

現時点で、私は一度もJuliaを触ったことがありません。
しかし、私自身あるプロジェクトでJuliaを使う機会を得たため、せっかくなら学びの記録を記事として残そうと思いました。

この記事では、**完全初心者がJuliaを学び始めるところから、環境構築・基本文法の習得**をまとめます。

同じく「これからJuliaを触ってみたい！」という方の参考になれば嬉しいです。


## 2.Juliaとは何か


Juliaは、2012年にMITの研究者たち（Jeff Bezanson、Alan Edelman、Stefan Karpinski、Viral B. Shah）によってMITで公開された、高性能な汎用プログラミング言語です。


その目的は明確で、「科学計算に必要な実行速度」と「動的言語の書きやすさ」を両立することです。

この設計思想は、よく引用される

> **「Cのように高速に動き、Pythonのように書きやすい」**
　というフレーズに集約されています。

 ### 高速性の理由

 Juliaは**LLVM（Low Level Virtual Machine）ベースのJITコンパイル**を採用しています。コードは実行時にネイティブコードへとコンパイルされ、静的型付き言語並の性能を発揮します。また、**型推論**と**特殊化**が組み合わさることで、関数や演算がデータ型に最適化され、無駄のないコードが生成されます。

 ### 主な特徴

- 高速な実行性能
コンパイル後はCやFortranに匹敵する速度。科学技術計算において大規模なデータ処理でも十分なパフォーマンスを発揮。

- 簡潔な文法
PythonやMATLABに似た構文で、数値演算や行列処理を直感的に記述可能。

- 多重ディスパッチ
引数の型の組み合わせに基づいて最適なメソッドを選択できるため、柔軟で拡張性の高いコード設計が可能。

- 並列・分散計算対応
標準ライブラリでマルチスレッドや分散処理をサポートし、HPC（High Performance Computing）用途にも適用可能。

- 数値計算に強い標準機能
複素数や行列、線形代数演算を言語レベルでサポート。BLASやLAPACKとの統合により、高速な数値処理が可能。

- 充実したパッケージエコシステム
機械学習（Flux.jl）、最適化（JuMP）、量子計算（Yao.jl）など、多分野にわたるライブラリが活発に開発されている。

### 応用分野

juliaは特に以下の分野で利用が広がっています。

- 数値シミュレーションや統計解析
- 機械学習・ディープラーニング
- 最適化問題
- 量子計算・物理シミュレーション

このように、Juliaは**高速性・表現力・科学計算向け機能**を兼ね備えた言語として、既存の「試作は動的言語、本番はC/Fortran」という二言語運用の課題を解消する存在として注目されています。






## 3.開発環境構築

### 私の環境

- OS: macOS Tahoe 26.0 beta
- CPU: Apple M3チップ
- パッケージマネージャ: Homebrew
- エディタ: VS Code（Julia拡張を入れて使う予定）
- 仮想環境: なし（Anaconda等は使わず、ローカルに直接インストール）

今回は、この環境で、Juliaをインストールし、VS Codeで動かせる状態にするまでをやってみます。

#### 1. HomebrewでJuliaを入れてみる

すでにHomebrewが入っているので、そのままコマンドを実行します。

```bash
brew install julia
```

インストール後、ちゃんと入ったかバージョンを確認します。

```bash
julia --version
```

出力結果：
```nginx
julia version 1.11.6
```
OKです。

#### 2. VS Codeの準備

VS Codeを起動し、左側の拡張機能アイコンから「Julia」と検索して、`Julia`拡張をインストールします。

#### 3. 動作確認

VS Codeで`hello.jl`というファイルを作成し、以下を入力して保存します。

```julia
println("Hello, Julia!")
```
初回実行時、次のようなメッセージが出ることがありますが、これは古いプロセス情報（pidfile）を削除しているだけのようなので問題ありません。
```
┌ Warning: attempting to remove probably stale pidfile
│   path = "/Users/⚪︎⚪︎⚪︎/.julia/compiled/v1.11/REPL/u0gqU_oGdfS.ji.pidfile"
└ @ FileWatching.Pidfile /usr/local/Cellar/julia/1.11.6/share/julia/stdlib/v1.11/FileWatching/src/pidfile.jl:249
Precompiling VSCodeServer...
  3 dependencies successfully precompiled in 51 seconds. 27 already precompiled.
Hello, Julia!
```
最後に、
```
Hello,julia!
```
と表示されれば、環境構築は完了です！

#### 4. 補足
今回私はローカルインストールにしましたが、
環境を汚したくない人はAnacondaなどの仮想環境でJuliaを入れる方法もあります。



## 4.基本文法の学習

Julia公式ドキュメントを参考に、基本文法の学習をしていきます。特に、数学演算に関する部分は、少し詳しく記します。


以下は[Julia公式ドキュメント](https://docs.julialang.org/en/v1/)の内容を参考にしながら書いています。

### はじめに - REPLを使った学習

Juliaの学習は対話型環境（REPL）から始めるのが最適だそうです。VS Code開いた手前、悲しいですが、ターミナルで進めていきます。

```bash
$ julia
```

```julia
julia> 1 + 2
3

julia> ans  # 直前の結果を参照
3
```

### 1. 変数と値の基本

#### 変数の宣言と代入

Juliaの変数は非常に柔軟です：

```julia
# 基本的な代入
x = 10
y = -3
name = "Hello World!"

# Unicode文字も使用可能
δ = 0.00001
π = 3.14159  # 注意：組み込み定数を上書き可能（推奨しない）
안녕하세요 = "Hello"

# 型注釈（オプション）
age::Int = 25
height::Float64 = 175.5
```

#### 変数の特徴

- **大文字小文字を区別**：`x`と`X`は別の変数
- **型の動的変更が可能**：
```julia
julia> x = 10
10

julia> x = "Hello World!"
"Hello World!"
```

- **LaTeX記号の入力**：`\delta`-*tab*で`δ`、`\alpha`-*tab*で`α`など

### 2.数値型と演算

#### 基本的な数値型

```julia
# 整数
a = 42
hex_val = 0x2a        # 16進数（42と同じ値）

# 浮動小数点数
pi_val = 3.14159
scientific = 2.5e-4

# 複素数
z = 1 + 2im
complex_calc = (3 + 4im) * (1 - 2im)

# 有理数
rational = 1//3
mixed = 2//3 + 1//6
```

#### 演算子の優先順位と結合性

Juliaの演算子は明確な優先順位が定められています。以下が公式の優先順位表（高い順）：

| カテゴリ | 演算子 | 結合性 |
|:---------|:-------|:-------|
| 構文 | `.` followed by `::` | 左 |
| 指数演算 | `^` | 右 |
| 単項演算子 | `+ - ! ~ ¬ √ ∛ ∜ ⋆ ± ∓ <: >:` | 右 |
| ビットシフト | `<< >> >>>` | 左 |
| 分数 | `//` | 左 |
| 乗除算 | `* / % & \ ÷` | 左 |
| 加減算 | `+ - \| ⊻` | 左 |
| 構文 | `: ..` | 左 |
| パイプ | `\|>` | 左 |
| 逆パイプ | `<\|` | 右 |
| 比較 | `> < >= <= == === != !== <:` | 非結合的 |
| 制御フロー | `&&` followed by `\|\|` followed by `?` | 右 |
| ペア | `=>` | 右 |
| 代入 | `= += -= *= /= //= \= ^= ÷= %= \|= &= ⊻= <<= >>= >>>=` | 右 |


#### 基本算術演算

```julia
# 優先順位の実例
julia> 10 + 5 * 2    # 乗算が先に実行される
20

julia> (10 + 5) * 2  # 括弧で優先順位を変更
30

julia> 2 * 3^2       # 指数が最優先（2 * 9 = 18）
18

julia> (2 * 3)^2     # 括弧で順序変更（6^2 = 36）
36

# 単項演算子の優先順位
julia> -2^2          # -(2^2) = -4
-4

julia> (-2)^2        # (-2)^2 = 4
4
```

#### 特殊な演算子

```julia
# 整数除算（\div-tabで入力）
julia> 7 ÷ 3
2

julia> 17 ÷ 5
3

# 剰余演算
julia> 17 % 5
2

julia> -17 % 5
-2

# 分数演算
julia> 1//3 + 2//5
11//15

julia> (3//4) * (8//9)
2//3

# 指数演算（右結合性）
julia> 2^3^2         # 2^(3^2) = 2^9 = 512
512

julia> (2^3)^2       # (2^3)^2 = 8^2 = 64
64
```

#### 数値リテラル係数（暗黙の乗算）

```julia
# 変数の前に数値を置くと乗算
julia> x = 3
3

julia> 2x
6

julia> 2x^2 - 3x + 1 # 多項式の美しい表記
10

julia> 1.5x^2 - .5x + 1
13.0

# 優先順位に注意
julia> 2^3x          # 2^(3x) として解釈
64

julia> 2x^3          # 2*(x^3) として解釈
54

# 括弧との組み合わせ
julia> 2(x-1)^2 - 3(x-1) + 1
3

julia> (x-1)x        # 括弧も係数になれる
6
```

#### 比較演算子

```julia
# 数値の比較
julia> 5 < 10
true

julia> 3.14 >= 3
true

julia> 42 == 42.0    # 異なる型でも値が等しければtrue
true

julia> 42 === 42.0   # 型も含めた厳密比較
false

# 文字列の比較
julia> "abc" == "abc"
true

julia> "abc" < "def"  # 辞書順
true
```

#### 連鎖比較

```julia
# 数学的な範囲表記が可能
julia> 1 ≤ 2 ≤ 3     # \le-tab で ≤
true

julia> 1 < 2 < 3
true

julia> 1 < 2 > 0     # 複雑な連鎖も可能
true

julia> x = 5
julia> 0 < x < 10
true

# 非結合性なので注意
julia> false == false == true
false

julia> (false == false) == true
true
```

#### Unicode比較演算子

```julia
# 様々なUnicode比較演算子が使用可能
julia> x ≠ y         # \ne-tab で ≠ (not equal)
julia> a ≤ b         # \le-tab で ≤ (less than or equal)
julia> a ≥ b         # \ge-tab で ≥ (greater than or equal)
julia> x ≈ y         # \approx-tab で ≈ (approximately equal)
```

#### 数学関数

```julia
# 平方根と冪乗
julia> sqrt(16)       # 平方根
4.0

julia> sqrt(2)
1.4142135623730951

julia> ∛(27)          # \cbrt-tab で立方根
3.0

julia> ∜(16)          # \fourthroot-tab で4乗根
2.0

# 絶対値と符号
julia> abs(-5)
5

julia> sign(-3.14)
-1.0

julia> sign(0)
0.0

julia> copysign(5, -3)  # 第2引数の符号を第1引数に適用
-5
```

#### 指数・対数関数

```julia
# 指数関数
julia> exp(1)         # 自然指数関数 e^x
2.718281828459045

julia> exp(0)
1.0

julia> expm1(0.001)   # exp(x) - 1 の高精度計算
0.0010005001667083846

# 対数関数
julia> log(ℯ)         # \euler-tab で ℯ (自然対数の底)
1.0

julia> log(2.718281828459045)
1.0

julia> log10(100)     # 常用対数
2.0

julia> log2(8)        # 2を底とする対数
3.0

julia> log(2, 8)      # 任意の底の対数 log_2(8)
3.0

julia> log1p(0.001)   # log(1 + x) の高精度計算
0.0009995003330835331
```

#### 三角関数

```julia
# 基本三角関数（ラジアン）
julia> sin(π/2)       # \pi-tab で π
1.0

julia> cos(π)
-1.0

julia> tan(π/4)
1.0

julia> sin(π/6)
0.49999999999999994

julia> sinpi(1/6)     # sin(π * x) の高精度計算
0.5

julia> cospi(1)       # cos(π * x) の高精度計算
-1.0

# 度数法での三角関数
julia> sind(90)       # 90度のサイン
1.0

julia> cosd(180)      # 180度のコサイン
-1.0

julia> tand(45)       # 45度のタンジェント
1.0

# 逆三角関数
julia> asin(1)        # アークサイン（ラジアン）
1.5707963267948966

julia> asind(1)       # アークサイン（度）
90.0

julia> atan(1, 1)     # atan2関数
0.7853981633974483
```

#### 双曲線関数

```julia
# 双曲線関数
julia> sinh(0)        # ハイパボリックサイン
0.0

julia> cosh(0)        # ハイパボリックコサイン
1.0

julia> tanh(0)        # ハイパボリックタンジェント
0.0

# 逆双曲線関数
julia> asinh(0)
0.0

julia> acosh(1)
0.0

julia> atanh(0)
0.0
```

#### 丸め関数

```julia
# 様々な丸め方法
julia> round(3.7)     # 最近接整数への丸め
4.0

julia> floor(3.7)     # 切り捨て
3.0

julia> ceil(3.2)      # 切り上げ
4.0

julia> trunc(3.7)     # 0に向かって切り捨て
3.0

julia> round(3.5)     # 偶数への丸め（デフォルト）
4.0

julia> round(4.5)
4.0

# 小数点以下の桁数指定
julia> round(π, digits=3)
3.142

julia> round(π, digits=0)
3.0
```

#### 特殊な数学関数

```julia
# ガンマ関数とその関連（SpecialFunctions.jlが必要）
# using SpecialFunctions
# gamma(5)      # 4! = 24
# lgamma(5)     # log(gamma(5))

# 誤差関数
# erf(1)        # 誤差関数
# erfc(1)       # 相補誤差関数

# ベッセル関数など（SpecialFunctions.jlパッケージで提供）

# 組み込みの特殊関数
julia> factorial(5)   # 階乗
120

julia> binomial(5, 2) # 二項係数
10
```
#### 短絡評価

```julia
# 論理AND（&&）- 左がfalseなら右を評価しない
julia> false && println("This won't print")
false

julia> true && println("This will print")
This will print
true

# 論理OR（||）- 左がtrueなら右を評価しない
julia> true || println("This won't print")
true

julia> false || println("This will print")
This will print
true

# 実用例：エラー回避
julia> x = 5
julia> x > 0 && sqrt(x)  # xが正の時のみ平方根を計算
2.23606797749979

julia> x = -1
julia> x > 0 && sqrt(x)  # xが負なら評価されない
false
```

#### ビット演算子

```julia
# ビット単位のAND、OR、XOR
julia> 5 & 3          # 101 & 011 = 001
1

julia> 5 | 3          # 101 | 011 = 111
7

julia> 5 ⊻ 3          # \xor-tab で ⊻（XOR）
6

julia> ~5             # ビット反転
-6

# ビットシフト
julia> 5 << 1         # 左シフト（5 * 2）
10

julia> 5 >> 1         # 右シフト（5 ÷ 2）
2

julia> -5 >>> 1       # 論理右シフト
9223372036854775805
```
#### 演算子は関数

```julia
# 演算子を関数として使用
julia> +(1, 2, 3)     # 中置記法と同じ
6

julia> *(2, 3, 4)
24

# 演算子を変数に代入
julia> my_add = +
+ (generic function with 166 methods)

julia> my_add(10, 20)
30

# map関数で演算子を使用
julia> map(+, [1, 2, 3], [4, 5, 6])
3-element Vector{Int64}:
 5
 7
 9
```

#### 演算子の優先順位確認

```julia
# 複雑な式の解析
julia> 2 + 3 * 4^2 / 8 - 1
7.0

# ステップバイステップ：
# 4^2 = 16        (指数が最優先)
# 3 * 16 = 48     (乗算)
# 48 / 8 = 6.0    (除算)
# 2 + 6.0 = 8.0   (加算)
# 8.0 - 1 = 7.0   (減算)

# 括弧で明示的に
julia> 2 + ((3 * (4^2)) / 8) - 1
7.0
```


### 3.文字列操作

#### 文字列の基本

```julia
# 文字列の作成
str1 = "Hello"
str2 = "World"
char = 'A'  # 文字（シングルクォート）

# 文字列結合
concatenated = str1 * " " * str2    # "Hello World"
interpolated = "$str1 $str2"        # 文字列補間（推奨）

# 複数行文字列
multiline = """
これは複数行の
文字列です。
インデントも保持されます。
"""
```

#### 高度な文字列操作

```julia
# 文字列補間の例
name = "Julia"
version = 1.10
message = "Welcome to $name v$version!"
calculation = "2 + 3 = $(2 + 3)"   # 式の評価も可能

# 文字列の操作
length("Hello")      # 5
uppercase("hello")   # "HELLO"
replace("Hello World", "World" => "Julia")  # "Hello Julia"

# 文字列の分割と結合
parts = split("a,b,c", ",")     # ["a", "b", "c"]
joined = join(["a", "b", "c"], "-")  # "a-b-c"
```

### 4.配列操作

#### 1次元配列（ベクトル）

```julia
# 配列の作成
arr1 = [1, 2, 3, 4, 5]
arr2 = Float64[1, 2, 3]        # 型指定
empty_arr = Int[]              # 空配列

# 範囲を使った配列生成
range_arr = collect(1:10)      # [1, 2, ..., 10]
step_range = collect(1:2:9)    # [1, 3, 5, 7, 9]
reverse_range = collect(10:-1:1) # [10, 9, ..., 1]

# 特殊な配列生成関数
zeros_arr = zeros(5)           # [0.0, 0.0, 0.0, 0.0, 0.0]
ones_arr = ones(Int, 3)        # [1, 1, 1]
fill_arr = fill("hello", 4)    # ["hello", "hello", "hello", "hello"]
```

#### インデックスアクセスとスライス

```julia
arr = [10, 20, 30, 40, 50]

# インデックスアクセス（1から始まる！！！！！）
first = arr[1]          # 10
last = arr[end]         # 50
second_last = arr[end-1] # 40

# スライス
slice1 = arr[2:4]       # [20, 30, 40]
slice2 = arr[1:2:end]   # [10, 30, 50]（ステップ指定）
reverse = arr[end:-1:1] # [50, 40, 30, 20, 10]

# 条件によるフィルタリング
filtered = arr[arr .> 25]  # [30, 40, 50]
```
インデックスが１から始まることには驚きです。

#### 2次元配列（行列）

```julia
# 行列の作成
matrix = [1 2 3; 4 5 6]        # 2×3行列
identity = [1 0; 0 1]          # 2×2単位行列

# 特殊行列
zeros_mat = zeros(3, 4)        # 3×4のゼロ行列
ones_mat = ones(2, 2)          # 2×2の1行列
identity_mat = I(3)            # 3×3単位行列（LinearAlgebra必要）

# 行列アクセス
julia> matrix[1, 2]            # 1行2列目: 2
julia> matrix[2, :]            # 2行目全体: [4, 5, 6]
julia> matrix[:, 1]            # 1列目全体: [1, 4]
```

#### 配列の変更操作

```julia
arr = [1, 2, 3]

# 要素の追加
push!(arr, 4)           # 末尾に追加: [1, 2, 3, 4]
pushfirst!(arr, 0)      # 先頭に追加: [0, 1, 2, 3, 4]
append!(arr, [5, 6])    # 複数要素追加: [0, 1, 2, 3, 4, 5, 6]

# 要素の削除
pop!(arr)               # 末尾を削除して返す: 6
popfirst!(arr)          # 先頭を削除して返す: 0
deleteat!(arr, 2)       # 2番目の要素を削除

# 要素の変更
arr[1] = 100           # 1番目を変更
arr[2:3] = [200, 300]  # 複数要素を一度に変更
```

### 5.関数定義

#### 基本的な関数定義

```julia
# 標準的な関数定義
function greet(name)
    return "Hello, $name!"
end

# 短縮記法（数学的表記）
square(x) = x^2
double(x) = 2x    # 乗算記号省略可能

# 複数の引数
function add_numbers(x, y)
    x + y  # return省略可能（最後の式が戻り値）
end

# 使用例
julia> greet("Julia")
"Hello, Julia!"

julia> square(5)
25

julia> add_numbers(3, 7)
10
```

#### 高度な関数機能

```julia
# デフォルト引数
function power(base, exponent=2)
    base^exponent
end

julia> power(3)     # 9（exponent=2がデフォルト）
julia> power(3, 3)  # 27

# キーワード引数
function create_person(name; age=25, city="Tokyo")
    "Name: $name, Age: $age, City: $city"
end

julia> create_person("Alice")
"Name: Alice, Age: 25, City: Tokyo"

julia> create_person("Bob", age=30, city="Osaka")
"Name: Bob, Age: 30, City: Osaka"

# 可変長引数
function sum_all(numbers...)
    total = 0
    for num in numbers
        total += num
    end
    return total
end

julia> sum_all(1, 2, 3, 4, 5)
15
```

#### 無名関数（ラムダ関数）

```julia
# 無名関数の定義
f = x -> x^2 + 2x + 1
g = (x, y) -> x + y

# 高階関数での使用
numbers = [1, 2, 3, 4, 5]
squares = map(x -> x^2, numbers)      # [1, 4, 9, 16, 25]
evens = filter(x -> x % 2 == 0, numbers)  # [2, 4]

# do記法（複数行の無名関数）
result = map(numbers) do x
    if x % 2 == 0
        x^2
    else
        x^3
    end
end
```

### 6.制御フロー

#### 条件分岐

```julia
# if-elseif-else文
function classify_number(x)
    if x > 0
        return "positive"
    elseif x < 0
        return "negative"
    else
        return "zero"
    end
end

# 三項演算子
sign = x >= 0 ? "non-negative" : "negative"

# 短絡評価
x > 0 && println("x is positive")    # xが正の時のみ実行
x == 0 || println("x is not zero")   # xが0でない時実行
```

#### ループ処理

```julia
# for ループ
for i in 1:5
    println("Count: $i")
end

# 配列の反復
fruits = ["apple", "banana", "orange"]
for fruit in fruits
    println("I like $fruit")
end

# enumerate（インデックスと値の両方）
for (index, fruit) in enumerate(fruits)
    println("$index: $fruit")
end

# while ループ
count = 1
while count <= 5
    println("While loop: $count")
    count += 1
end

# ネストしたループ
for i in 1:3
    for j in 1:2
        println("($i, $j)")
    end
end
```

#### 内包表記

```julia
# 配列内包表記
squares = [x^2 for x in 1:10]               # [1, 4, 9, ..., 100]
evens = [x for x in 1:20 if x % 2 == 0]     # [2, 4, 6, ..., 20]

# 条件付き内包表記
positive_squares = [x^2 for x in -5:5 if x > 0]  # [1, 4, 9, 16, 25]

# 辞書内包表記
squares_dict = Dict(x => x^2 for x in 1:5)  # Dict(1=>1, 2=>4, ...)

# ジェネレータ式（メモリ効率的）
sum_of_squares = sum(x^2 for x in 1:1000000)  # 配列を作らずに計算
```

### 7.型システム

#### 型の確認と変換

```julia
# 型の確認
julia> typeof(42)
Int64

julia> typeof(3.14)
Float64

julia> typeof("Hello")
String

# 型チェック
julia> 42 isa Int
true

julia> 3.14 isa Number
true

julia> "Hello" isa String
true

# 型変換
int_val = Int(3.14)         # 3
float_val = Float64(42)     # 42.0
string_val = string(123)    # "123"
parse_int = parse(Int, "456")  # 456
```

#### 型注釈と型安定性

```julia
# 関数の型注釈
function typed_function(x::Int, y::Float64)::Float64
    return x * y
end

# 変数の型注釈
age::Int = 25
height::Float64 = 175.5

# 型パラメータを持つ関数
function process_array(arr::Vector{T}) where T <: Number
    return sum(arr) / length(arr)
end
```

### 8.データ構造

#### タプル（不変）

```julia
# タプルの作成
point = (3, 4)
person = ("Alice", 25, "Engineer")

# 分割代入
x, y = point
name, age, job = person

# 名前付きタプル
coord = (x=10, y=20, z=5)
julia> coord.x  # 10
julia> coord.y  # 20

# タプルの操作
length(point)    # 2
point[1]         # 3（1から始まるインデックス）
```

#### 辞書（Dictionary）

```julia
# 辞書の作成
person = Dict("name" => "Alice", "age" => 30, "city" => "Tokyo")
scores = Dict(:math => 95, :english => 88, :science => 92)

# 辞書の操作
julia> person["name"]        # "Alice"
julia> person["age"] = 31    # 値の更新
julia> person["job"] = "Engineer"  # 新しいキーの追加

# キーと値の取得
keys(person)    # キーの一覧
values(person)  # 値の一覧

# 安全なアクセス
get(person, "salary", 0)     # キーがなければ0を返す
haskey(person, "name")       # キーの存在チェック

# 辞書の反復
for (key, value) in person
    println("$key: $value")
end
```

### 9.モジュールとパッケージ

#### 標準ライブラリの使用

```julia
# 統計関数
using Statistics
data = [1, 2, 3, 4, 5]
avg = mean(data)      # 平均値
med = median(data)    # 中央値
std_dev = std(data)   # 標準偏差

# 線形代数
using LinearAlgebra
A = [1 2; 3 4]
B = [5 6; 7 8]
product = A * B       # 行列の積
det_A = det(A)        # 行列式
eigenvals = eigvals(A)  # 固有値

# 部分的インポート
import LinearAlgebra: dot, norm
dot_product = dot([1, 2], [3, 4])  # 内積
vector_norm = norm([3, 4])         # ベクトルのノルム
```

#### 自作モジュール

```julia
# モジュールの定義
module MathUtils
    export square, cube, factorial_iter
    
    # 公開関数
    square(x) = x^2
    cube(x) = x^3
    
    # 反復的階乗（内部関数）
    function factorial_iter(n)
        result = 1
        for i in 1:n
            result *= i
        end
        return result
    end
    
    # 非公開関数
    helper_function(x) = x + 1
end

# モジュールの使用
using .MathUtils
julia> square(5)    # 25
julia> cube(3)      # 27
```

### 10.エラーハンドリング

#### try-catch文

```julia
# 基本的なエラーハンドリング
function safe_divide(a, b)
    try
        result = a / b
        return result
    catch e
        if isa(e, DivideError)
            println("Error: Division by zero!")
            return nothing
        else
            println("Unexpected error: $e")
            rethrow(e)  # 他のエラーは再発生
        end
    end
end

julia> safe_divide(10, 2)   # 5.0
julia> safe_divide(10, 0)   # Error: Division by zero!

# finally節
function read_file_safely(filename)
    file = nothing
    try
        file = open(filename, "r")
        content = read(file, String)
        return content
    catch e
        println("Failed to read file: $e")
        return nothing
    finally
        if file !== nothing
            close(file)  # ファイルを確実にクローズ
        end
    end
end
```

#### カスタム例外

```julia
# カスタム例外の定義
struct NegativeNumberError <: Exception
    message::String
end

function square_root_positive(x)
    if x < 0
        throw(NegativeNumberError("Square root of negative number: $x"))
    end
    return sqrt(x)
end

# 使用例
try
    result = square_root_positive(-4)
catch e::NegativeNumberError
    println("Custom error caught: $(e.message)")
end
```

## 4.まとめ
今回はjuliaをインストールして、基本文法をざっくり確認しました。正直、まだほとんど覚えていませんが、今後実際に使いながら、少しずつ身につけていければと思います。

触ってみた印象としては、読みやすくて直感的に書けるのが嬉しいポイントでした。

これからJuliaを始める方にとって、この記事が最初の一歩の参考になれば嬉しいです。

## 参考資料

- [Julia公式ドキュメント](https://docs.julialang.org/en/v1/)
- [Julia Data Science](https://juliadatascience.io/)
- [Wikipedia(Julia)](https://en.wikipedia.org/wiki/Julia_(programming_language))
