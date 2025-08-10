## 1.はじめに
現在B2で、研究室に配属されたことをきっかけに初めてQiitaに投稿します。
今回は、自分の学びを整理しつつ「量子計算入門」としてまとめてみました。

この記事では、量子コンピュータの基礎となる量子ビットの表現方法やブラケット記法、  
そして代表的な量子ゲートまでを解説していきます。

内容について誤りやわかりにくい点があれば、ぜひコメントで指摘していただけると嬉しいです。

## 2. 量子力学と線形代数のミニマム

### 2.1 量子状態と複素ベクトル

量子力学では、システムの状態は**複素ベクトル空間**の単位ベクトルで表現されます。1量子ビットの場合、2次元の複素ベクトル空間を使います。

量子状態は以下のように表現されます：

```math
|\psi\rangle = \begin{pmatrix}
c_1 \\
\vdots \\
c_d
\end{pmatrix}
```

ここで、$c_i$は複素数の**確率振幅**と呼ばれ、$|c_i|^2$が各状態の観測確率を表します。

### 2.2 ブラケット記法

量子力学では**ブラケット記法**（Dirac記法）が標準的に使用されます：

- **ケット**：$|\psi\rangle$ は列ベクトルを表します
- **ブラ**：$\langle\psi|$ は行ベクトル（ケットの複素共役転置）を表します

```math
|\psi\rangle = \begin{pmatrix}
c_1 \\
\vdots \\
c_d
\end{pmatrix}, \quad 
\langle\psi| = (c_1^* \cdots c_d^*)
```

### 2.3 内積と外積

2つの量子状態の**内積**は以下のように定義されます：

```math
\langle\phi|\psi\rangle = \sum_k \phi_k^* \psi_k
```

**外積**は行列を作り出します：

```math
|\phi\rangle\langle\psi| = \begin{pmatrix}
\phi_1 \psi_1^* & \cdots & \phi_1 \psi_d^* \\
\vdots & \ddots & \vdots \\
\phi_d \psi_1^* & \cdots & \phi_d \psi_d^*
\end{pmatrix}
```

### 2.4 規格化条件

量子状態は必ず**規格化**されている必要があります：

```math
\langle\psi|\psi\rangle = \|\psi\rangle\|^2 = 1
```

これにより、すべての測定確率の合計が1になることが保証されます。

## 3. 1量子ビットの表現

### 3.1 基底状態の詳細

1量子ビットの**計算基底**は以下の2つの状態で構成されます：

```math
|0\rangle = \begin{pmatrix}
1 \\
0
\end{pmatrix}, \quad
|1\rangle = \begin{pmatrix}
0 \\
1
\end{pmatrix}
```

これらの状態は**正規直交基底**を形成します：
- **正規性**: $\langle 0|0\rangle = \langle 1|1\rangle = 1$
- **直交性**: $\langle 0|1\rangle = \langle 1|0\rangle = 0$

古典ビットとは異なり、量子ビットはこれらの基底状態の**線形結合（重ね合わせ）**を取ることができます。これが量子計算の根本的な特徴です。

### 3.2 一般的な量子状態と確率振幅

任意の1量子ビット状態は以下のように表現できます：

```math
|\psi\rangle = \alpha|0\rangle + \beta|1\rangle = \begin{pmatrix}
\alpha \\
\beta
\end{pmatrix}
```

ここで重要なのは以下の点です：

**確率振幅の意味**:
- $\alpha, \beta$は複素数で、**確率振幅**と呼ばれます
- $|\alpha|^2$は測定時に$|0\rangle$が観測される確率
- $|\beta|^2$は測定時に$|1\rangle$が観測される確率

**規格化条件**:
```math
|\alpha|^2 + |\beta|^2 = 1
```

この条件により、すべての測定確率の合計が1になることが保証されます。

**位相の重要性**:
確率振幅は複素数なので、各振幅には**位相**（偏角）が含まれます。この位相が量子干渉効果を生み出し、量子アルゴリズムの基礎となります。

### 3.3 重要な特殊状態

量子計算では以下のような特殊な状態が頻繁に使用されます：

**X基底の状態**:
```math
|+\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\1\end{pmatrix}
```

```math
|-\rangle = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\-1\end{pmatrix}
```

**Y基底の状態**:
```math
|+i\rangle = \frac{1}{\sqrt{2}}(|0\rangle + i|1\rangle) = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\i\end{pmatrix}
```

```math
|-i\rangle = \frac{1}{\sqrt{2}}(|0\rangle - i|1\rangle) = \frac{1}{\sqrt{2}}\begin{pmatrix}1\\-i\end{pmatrix}
```

これらの状態は**相互に直交**しており、それぞれ異なる測定基底を構成します。

**内積の計算例**:
```math
\langle 0|+\rangle = \frac{1}{\sqrt{2}}
```
```math
\langle 1|-\rangle = -\frac{1}{\sqrt{2}}
```
```math
\langle +|-\rangle = 0 \quad \text{（直交）}
```

### 3.4 ブロッホ球表現の詳細

**ブロッホ球**は1量子ビット状態を3次元球面上の点として視覚化する強力な方法です。

**パラメータ表現**:
```math
|\psi\rangle = \cos\left(\frac{\theta}{2}\right)|0\rangle + e^{i\phi}\sin\left(\frac{\theta}{2}\right)|1\rangle
```

**球面座標との対応**:
- $\theta$ (0 ≤ θ ≤ π): 極角（z軸からの角度）
- $\phi$ (0 ≤ φ < 2π): 方位角（x-y平面での角度）

**特殊な点の位置**:
- **北極** (θ=0): $|0\rangle$状態
- **南極** (θ=π): $|1\rangle$状態
- **x軸正方向** (θ=π/2, φ=0): $|+\rangle$状態
- **x軸負方向** (θ=π/2, φ=π): $|-\rangle$状態
- **y軸正方向** (θ=π/2, φ=π/2): $|+i\rangle$状態
- **y軸負方向** (θ=π/2, φ=3π/2): $|-i\rangle$状態

**ブロッホ球の重要性**:
1. 量子状態の幾何学的直感を提供
2. 量子演算を球面上の回転として理解可能
3. 異なる測定基底の関係を視覚的に把握

**グローバル位相の無視**:
```math
e^{i\gamma}|\psi\rangle
```
のような全体位相$e^{i\gamma}$は測定では区別できないため、ブロッホ球では同一点として扱われます。

## 4. 1量子ビットの操作

### 4.1 ユニタリ演算子の詳細

量子状態の時間発展は**ユニタリ演算子**$U$によって記述されます：

```math
|\psi'\rangle = U|\psi\rangle
```

**ユニタリ性の条件**:
```math
U^\dagger U = I
```

ここで$U^\dagger$は$U$の**エルミート共役**（複素共役転置）です。

**ユニタリ演算子の重要な性質**:
1. **可逆性**: $U^{-1} = U^\dagger$なので、すべての量子操作は原理的に可逆
2. **規格化保存**: $\langle\psi'|\psi'\rangle = \langle\psi|U^\dagger U|\psi\rangle = \langle\psi|\psi\rangle = 1$
3. **距離保存**: 2つの状態間の内積を保持

**エルミート共役の計算**:
行列
```math
A = \begin{pmatrix}a_{11} & \cdots & a_{1d}\\\vdots & \ddots & \vdots\\a_{d1} & \cdots & a_{dd}\end{pmatrix}
```
に対して：

```math
A^\dagger = \begin{pmatrix}a_{11}^* & \cdots & a_{d1}^*\\\vdots & \ddots & \vdots\\a_{1d}^* & \cdots & a_{dd}^*\end{pmatrix}
```

### 4.2 パウリ演算子の詳細解析

**パウリ行列の定義**:
```math
I = \begin{pmatrix}1 & 0\\0 & 1\end{pmatrix}, \quad
X = \begin{pmatrix}0 & 1\\1 & 0\end{pmatrix}
```

```math
Y = \begin{pmatrix}0 & -i\\i & 0\end{pmatrix}, \quad
Z = \begin{pmatrix}1 & 0\\0 & -1\end{pmatrix}
```

**X演算子（ビットフリップ）の作用**:
```math
X|0\rangle = \begin{pmatrix}0 & 1\\1 & 0\end{pmatrix}\begin{pmatrix}1\\0\end{pmatrix} = \begin{pmatrix}0\\1\end{pmatrix} = |1\rangle
```

```math
X|1\rangle = \begin{pmatrix}0 & 1\\1 & 0\end{pmatrix}\begin{pmatrix}0\\1\end{pmatrix} = \begin{pmatrix}1\\0\end{pmatrix} = |0\rangle
```

**Z演算子（位相フリップ）の作用**:
```math
Z|0\rangle = |0\rangle, \quad Z|1\rangle = -|1\rangle
```

**重ね合わせ状態への作用**:
```math
X|+\rangle = X \cdot \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) = \frac{1}{\sqrt{2}}(|1\rangle + |0\rangle) = |+\rangle
```

```math
Z|+\rangle = Z \cdot \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle) = \frac{1}{\sqrt{2}}(|0\rangle - |1\rangle) = |-\rangle
```

**パウリ演算子の代数的性質**:
- $X^2 = Y^2 = Z^2 = I$（べき等性）
- $Y = iXZ$（3つの演算子の関係）
- $XZ = -ZX$（反可換関係）

これらの性質により、パウリ演算子は**SU(2)群**の生成元となります。

### 4.3 Hadamard演算子の深い理解

**Hadamard演算子**は量子計算における最も重要な演算子の一つです：

```math
H = \frac{1}{\sqrt{2}}\begin{pmatrix}1 & 1\\1 & -1\end{pmatrix}
```

**基底変換としての役割**:
- $H|0\rangle = |+\rangle$: 計算基底から+/-基底への変換
- $H|1\rangle = |-\rangle$
- $H|+\rangle = |0\rangle$: 逆変換
- $H|-\rangle = |1\rangle$

**重ね合わせの生成**:
Hadamard演算子は確定的な古典状態$|0\rangle$から等確率の重ね合わせ状態を生成します：

```math
H|0\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)
```

この状態を測定すると、50%の確率で0、50%の確率で1が観測されます。

**Hadamardの幾何学的解釈**:
ブロッホ球上では、Hadamard演算子はx軸とz軸の間の45度回転（π回転）に対応します。

### 4.4 位相ゲートの詳細

**S演算子（π/2位相ゲート）**:
```math
S = \begin{pmatrix}1 & 0\\0 & i\end{pmatrix}
```

作用：
- $S|0\rangle = |0\rangle$
- $S|1\rangle = i|1\rangle$ （位相にiを乗算）

**T演算子（π/4位相ゲート）**:
```math
T = \begin{pmatrix}e^{-i\pi/8} & 0\\0 & e^{i\pi/8}\end{pmatrix}
```

T演算子は**普遍量子計算**において重要な役割を果たします。

### 4.5 回転演算子の詳細解析

パウリ演算子$A$（$A = X, Y, Z$）に対する**回転演算子**：

```math
R_A(\theta) = e^{-i(\theta/2)A} = \cos(\theta/2)I - i\sin(\theta/2)A
```

この指数表現は**行列の指数関数**で、Taylor展開から導出されます。

**Z軸周りの回転**:
```math
R_Z(\theta) = e^{-i(\theta/2)Z} = \begin{pmatrix}e^{-i\theta/2} & 0\\0 & e^{i\theta/2}\end{pmatrix}
```

**X軸周りの回転**:
```math
R_X(\theta) = e^{-i(\theta/2)X} = \begin{pmatrix}\cos(\theta/2) & -i\sin(\theta/2)\\-i\sin(\theta/2) & \cos(\theta/2)\end{pmatrix}
```

**Y軸周りの回転**:
```math
R_Y(\theta) = e^{-i(\theta/2)Y} = \begin{pmatrix}\cos(\theta/2) & -\sin(\theta/2)\\\sin(\theta/2) & \cos(\theta/2)\end{pmatrix}
```

**回転の物理的意味**:
- $R_Z(\theta)$: ブロッホ球のz軸周りにθだけ回転
- $R_X(\theta)$: ブロッホ球のx軸周りにθだけ回転  
- $R_Y(\theta)$: ブロッホ球のy軸周りにθだけ回転

### 4.6 任意回転のオイラー分解

**重要な定理**: 任意の1量子ビットユニタリ演算子は以下のように分解できます：

```math
U = e^{i\phi}R_z(\alpha)R_x(\beta)R_z(\gamma)
```

この分解により、任意の量子演算を3つの基本的な回転操作の組み合わせとして実装できます。

**分解の意味**:
1. 最初のZ回転（角度γ）
2. X回転（角度β）
3. 最後のZ回転（角度α）
4. 全体位相$e^{i\phi}$

この分解は量子回路の最適化や、物理的実装において非常に重要です。

## 5. 量子状態の測定

### 5.1 射影測定の数学的記述

量子測定は**射影演算子**$\{P_k\}$を用いて記述されます。これらの演算子は以下の条件を満たします：

**完全性条件**:
```math
\sum_k P_k = I
```

**直交性条件**:
```math
P_k P_l = \delta_{kl} P_k
```

**エルミート性**:
```math
P_k^\dagger = P_k
```

### 5.2 計算基底での測定

最も基本的な測定は**計算基底**$\{|0\rangle, |1\rangle\}$での測定です。

**射影演算子**:
```math
P_0 = |0\rangle\langle0| = \begin{pmatrix}1 & 0\\0 & 0\end{pmatrix}
```

```math
P_1 = |1\rangle\langle1| = \begin{pmatrix}0 & 0\\0 & 1\end{pmatrix}
```

**測定確率の計算**:
状態$|\psi\rangle = \alpha|0\rangle + \beta|1\rangle$に対して：

```math
p_0 = \langle\psi|P_0|\psi\rangle = |\langle0|\psi\rangle|^2 = |\alpha|^2
```

```math
p_1 = \langle\psi|P_1|\psi\rangle = |\langle1|\psi\rangle|^2 = |\beta|^2
```

**測定後の状態**:
測定結果が$k$だった場合、システムは以下の状態に**収縮**します：

```math
|\psi'\rangle = \frac{P_k|\psi\rangle}{\|P_k|\psi\rangle\|}
```

**具体例**:
$|\psi\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$を計算基底で測定した場合：
- 50%の確率で0を観測 → 状態は$|0\rangle$に収縮
- 50%の確率で1を観測 → 状態は$|1\rangle$に収縮

### 5.3 異なる基底での測定

**+/-基底での測定**:
```math
P_+ = |+\rangle\langle+| = \frac{1}{2}\begin{pmatrix}1 & 1\\1 & 1\end{pmatrix}
```

```math
P_- = |-\rangle\langle-| = \frac{1}{2}\begin{pmatrix}1 & -1\\-1 & 1\end{pmatrix}
```

**測定確率の計算例**:
状態$|0\rangle$を+/-基底で測定：
```math
p_+ = |\langle+|0\rangle|^2 = \left|\frac{1}{\sqrt{2}}\right|^2 = \frac{1}{2}
```

```math
p_- = |\langle-|0\rangle|^2 = \left|\frac{1}{\sqrt{2}}\right|^2 = \frac{1}{2}
```

### 5.4 期待値と物理量

**期待値の定義**:
物理量（観測可能量）$A$の期待値は：

```math
\langle A \rangle = \langle\psi|A|\psi\rangle
```

**Z演算子の期待値**:
```math
\langle Z \rangle = \langle\psi|Z|\psi\rangle = p_0 \times (+1) + p_1 \times (-1) = p_0 - p_1 = |\alpha|^2 - |\beta|^2
```

この値は-1から+1の範囲を取り、量子状態の「z方向の傾き」を表します。

**パウリ演算子の期待値とブロッホベクトル**:
3つのパウリ演算子の期待値$(⟨X⟩, ⟨Y⟩, ⟨Z⟩)$は**ブロッホベクトル**を構成し、ブロッホ球上の点の座標を与えます。

### 5.5 測定の不可逆性

量子測定には以下の重要な特徴があります：

1. **確率的性質**: 同一の状態を測定しても、結果は確率的に決まる
2. **状態の破壊**: 測定により重ね合わせ状態は基底状態に収縮
3. **情報の獲得と喪失**: 測定により古典情報を得るが、量子情報は失われる

この性質により、量子状態をコピーすることはできません（**量子複製不可能定理**）。

## 6. 複数量子ビット

### 6.1 テンソル積

複数の量子システムは**テンソル積**で結合されます。2つの量子状態$|\psi\rangle$と$|\phi\rangle$の結合状態は：

```math
|\psi\rangle \otimes |\phi\rangle = |\psi\rangle|\phi\rangle
```

と表記されます。

### 6.2 2量子ビットの基底

2量子ビットシステムの計算基底は4つの状態からなります：

```math
|00\rangle, |01\rangle, |10\rangle, |11\rangle
```

これらはテンソル積で以下のように表現されます：

```math
|00\rangle = |0\rangle \otimes |0\rangle = \begin{pmatrix}1\\0\\0\\0\end{pmatrix}
```

```math
|01\rangle = |0\rangle \otimes |1\rangle = \begin{pmatrix}0\\1\\0\\0\end{pmatrix}
```

```math
|10\rangle = |1\rangle \otimes |0\rangle = \begin{pmatrix}0\\0\\1\\0\end{pmatrix}
```

```math
|11\rangle = |1\rangle \otimes |1\rangle = \begin{pmatrix}0\\0\\0\\1\end{pmatrix}
```

### 6.3 一般的な2量子ビット状態

任意の2量子ビット状態は以下のように表現されます：

```math
|\psi\rangle_{AB} = c_{00}|00\rangle + c_{01}|01\rangle + c_{10}|10\rangle + c_{11}|11\rangle
```

```math
= \begin{pmatrix}c_{00}\\c_{01}\\c_{10}\\c_{11}\end{pmatrix}
```

規格化条件は$|c_{00}|^2 + |c_{01}|^2 + |c_{10}|^2 + |c_{11}|^2 = 1$です。

### 6.4 テンソル積の計算規則

以下の規則が成り立ちます：

```math
\left(\sum_i a_i|i\rangle\right) \otimes \left(\sum_j b_j|j\rangle\right) = \sum_{ij} a_i b_j |i\rangle \otimes |j\rangle
```

```math
(A \otimes B)(|\psi\rangle \otimes |\phi\rangle) = (A|\psi\rangle) \otimes (B|\phi\rangle)
```

## 7. 2量子ビットの操作

### 7.1 CNOTゲート

最も重要な2量子ビット演算子は**CNOT（制御NOT）ゲート**です：

```math
\text{CNOT} = |0\rangle\langle0| \otimes I + |1\rangle\langle1| \otimes X
```

行列表現では：

```math
\text{CNOT} = \begin{pmatrix}
1 & 0 & 0 & 0 \\
0 & 1 & 0 & 0 \\
0 & 0 & 0 & 1 \\
0 & 0 & 1 & 0
\end{pmatrix}
```

CNOTゲートの動作：
- $|00\rangle \rightarrow |00\rangle$
- $|01\rangle \rightarrow |01\rangle$
- $|10\rangle \rightarrow |11\rangle$
- $|11\rangle \rightarrow |10\rangle$

制御量子ビット（1番目）が$|1\rangle$の時のみ、標的量子ビット（2番目）にXゲートが適用されます。

### 7.2 ベル状態の生成

CNOTゲートとHadamardゲートを組み合わせることで**ベル状態**を生成できます：

```math
\text{CNOT}(H \otimes I)|00\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)
```

計算過程：
1. $H|0\rangle = \frac{1}{\sqrt{2}}(|0\rangle + |1\rangle)$
2. $(H \otimes I)|00\rangle = \frac{1}{\sqrt{2}}(|00\rangle + |10\rangle)$
3. $\text{CNOT}\frac{1}{\sqrt{2}}(|00\rangle + |10\rangle) = \frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$

### 7.3 量子もつれ

ベル状態$\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle)$は**量子もつれ状態**の例です。この状態は単一の量子ビット状態のテンソル積では表現できません。

量子もつれ状態では、一方の量子ビットを測定すると、他方の量子ビットの状態が瞬時に決定されます：

- 1番目の量子ビットで$|0\rangle$を観測 → 2番目も必ず$|0\rangle$
- 1番目の量子ビットで$|1\rangle$を観測 → 2番目も必ず$|1\rangle$

### 7.4 異なる基底での測定

ベル状態を$\{|+\rangle, |-\rangle\}$基底で表現すると：

```math
\frac{1}{\sqrt{2}}(|00\rangle + |11\rangle) = \frac{1}{\sqrt{2}}(|++\rangle + |--\rangle)
```

この表現からも量子もつれの性質が確認できます。

## 8. 古典計算との違い

### 8.1 状態の表現

- **古典ビット**: $n$ビットの状態は$2^n$通りの中の1つ
- **量子ビット**: $n$量子ビットの状態は$2^n$個の複素振幅で表現（指数的に多くの情報を保持）

### 8.2 並列性

量子コンピュータは重ね合わせ状態により、すべての可能な入力に対して同時に計算を実行できます。例えば、$n$量子ビットの重ね合わせ状態に関数$f$を適用すると：

```math
\sum_{x \in \{0,1\}^n} \alpha_x |x\rangle \rightarrow \sum_{x \in \{0,1\}^n} \alpha_x |x\rangle|f(x)\rangle
```

すべての$x$に対する$f(x)$が同時に計算されます。

### 8.3 干渉効果

量子アルゴリズムでは、確率振幅の**干渉効果**を利用します：
- **構成的干渉**: 正しい答えの確率振幅を増大
- **相殺的干渉**: 間違った答えの確率振幅を減少

### 8.4 測定の不可逆性

量子測定は不可逆的で、重ね合わせ状態を破壊します。これにより、量子情報は古典情報よりも扱いが困難ですが、同時に新しい計算の可能性を開きます。

## まとめ

今回は、量子ビットの表現方法やブラケット記法、そしてブロッホ球のイメージについて見てきました。  最近は量子コンピュータがニュースで話題になることも増えていますが、その根底にあるこうした基本概念を理解しておくと、話題をより深く楽しめるはずです。  

この記事が、量子計算の基礎に対する興味の入り口になっていれば嬉しいです。  





