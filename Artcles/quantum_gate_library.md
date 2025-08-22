
# 量子ゲート変換図鑑

これは私自身の学習のためにまとめた「量子ゲート変換図鑑」です。  
量子回路でよく使うゲートの定義や分解、交換関係などを整理し、数式と行列表記を併記しました。

## 1. 基本定義

### Pauliゲート

**Xゲート（ビット反転）**

```math
X|0\rangle = |1\rangle, \quad X|1\rangle = |0\rangle
```

```math
X = \begin{pmatrix} 0 & 1 \\ 1 & 0 \end{pmatrix}
```

Xはビット反転、古典のNOTゲートに対応。量子状態の重ね合わせも反転する。

**Yゲート**

```math
Y|0\rangle = i|1\rangle, \quad Y|1\rangle = -i|0\rangle
```

```math
Y = \begin{pmatrix} 0 & -i \\ i & 0 \end{pmatrix}
```

Yは位相付きビット反転。X軸とZ軸の回転の合成として理解できる。

**Zゲート（位相反転）**

```math
Z|0\rangle = |0\rangle, \quad Z|1\rangle = -|1\rangle
```

```math
Z = \begin{pmatrix} 1 & 0 \\ 0 & -1 \end{pmatrix}
```

Zは|1⟩の位相だけ反転。重ね合わせ状態で位相干渉を制御するのに重要。

---

### Hadamardゲート

**Hadamardゲート**

```math
H|0\rangle = \frac{|0\rangle + |1\rangle}{\sqrt{2}}, \quad H|1\rangle = \frac{|0\rangle - |1\rangle}{\sqrt{2}}
```

```math
H = \frac{1}{\sqrt{2}}\begin{pmatrix} 1 & 1 \\ 1 & -1 \end{pmatrix}
```

Hadamardは重ね合わせを作る基本ゲート。計算基底と対角基底を相互変換する。

---

### 位相ゲート

**Sゲート（π/2位相ゲート）**

```math
S|0\rangle = |0\rangle, \quad S|1\rangle = i|1\rangle
```

```math
S = \begin{pmatrix} 1 & 0 \\ 0 & i \end{pmatrix}
```

**回転ゲートとの関係：**
```math
S = e^{i\pi/4} R_z(\pi/2)
```

ここで、グローバル位相 $e^{i\pi/4}$ を含む表現。

**Tゲート（π/4位相ゲート）**

```math
T|0\rangle = |0\rangle, \quad T|1\rangle = e^{i\pi/4}|1\rangle
```

```math
T = \begin{pmatrix} 1 & 0 \\ 0 & e^{i\pi/4} \end{pmatrix}
```

**回転ゲートとの関係：**
```math
T = e^{i\pi/8} R_z(\pi/4)
```

Tはπ/4の位相をかける。量子計算の普遍性に不可欠。

---

### 回転ゲート

**X軸回転**

```math
R_x(\theta) = e^{-i\frac{\theta}{2}X} = \cos\frac{\theta}{2}I - i\sin\frac{\theta}{2}X
```

```math
R_x(\theta) = \begin{pmatrix} \cos\frac{\theta}{2} & -i\sin\frac{\theta}{2} \\ -i\sin\frac{\theta}{2} & \cos\frac{\theta}{2} \end{pmatrix}
```

Bloch球のX軸周りの回転。θ=πでXゲートになる。

**Y軸回転**

```math
R_y(\theta) = e^{-i\frac{\theta}{2}Y} = \cos\frac{\theta}{2}I - i\sin\frac{\theta}{2}Y
```

```math
R_y(\theta) = \begin{pmatrix} \cos\frac{\theta}{2} & -\sin\frac{\theta}{2} \\ \sin\frac{\theta}{2} & \cos\frac{\theta}{2} \end{pmatrix}
```

Bloch球のY軸周りの回転。実行列になっているのが特徴。

**Z軸回転**

```math
R_z(\theta) = e^{-i\frac{\theta}{2}Z} = \cos\frac{\theta}{2}I - i\sin\frac{\theta}{2}Z
```

```math
R_z(\theta) = \begin{pmatrix} e^{-i\theta/2} & 0 \\ 0 & e^{i\theta/2} \end{pmatrix}
```

Bloch球のZ軸周りの回転。位相のみを変化させる。


## 2. 多量子ビットゲート

### CNOTゲート（制御X）

```math
\text{CNOT}|00\rangle = |00\rangle, \quad \text{CNOT}|01\rangle = |01\rangle
```
```math
\text{CNOT}|10\rangle = |11\rangle, \quad \text{CNOT}|11\rangle = |10\rangle
```

```math
\text{CNOT} = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \\ 0 & 0 & 1 & 0 \end{pmatrix}
```

### SWAPゲート

```math
\text{SWAP}|00\rangle = |00\rangle, \quad \text{SWAP}|01\rangle = |10\rangle
```
```math
\text{SWAP}|10\rangle = |01\rangle, \quad \text{SWAP}|11\rangle = |11\rangle
```

```math
\text{SWAP} = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 0 & 1 \end{pmatrix}
```

**SWAP分解（CNOTで）：**
```math
\text{SWAP} = \text{CNOT}_{12} \cdot \text{CNOT}_{21} \cdot \text{CNOT}_{12}
```

### CZゲート（制御Z）

```math
\text{CZ}|00\rangle = |00\rangle, \quad \text{CZ}|01\rangle = |01\rangle
```
```math
\text{CZ}|10\rangle = |10\rangle, \quad \text{CZ}|11\rangle = -|11\rangle
```

```math
\text{CZ} = \begin{pmatrix} 1 & 0 & 0 & 0 \\ 0 & 1 & 0 & 0 \\ 0 & 0 & 1 & 0 \\ 0 & 0 & 0 & -1 \end{pmatrix}
```

**CNOTとの変換：**
```math
\text{CZ} = (I \otimes H) \cdot \text{CNOT} \cdot (I \otimes H)
```

### Toffoliゲート（CCX）

```math
\text{CCX}|abc\rangle = |ab(c \oplus ab)\rangle
```

**標準分解（Shende et al., 2009）：**
```
CCX = H₃ · CNOT₂₃ · T₃† · CNOT₁₃ · T₃ · CNOT₂₃ · T₃† · CNOT₁₃ · T₁ · T₂ · T₃ · CNOT₁₂ · H₃ · T₁† · CNOT₁₂
```



## 3. 重要な変換関係式

### Hadamard変換による基底変換

```math
HXH = Z
```
```math
HZH = X  
```
```math
HYH = -Y
```

Hadamardは計算基底⇄対角基底の変換を行う。XとZが交換され、Yは符号が変わる。

### 位相ゲートの周期性

```math
S^2 = Z, \quad S^4 = I
```
```math
T^2 = S, \quad T^4 = Z, \quad T^8 = I
```

**逆ゲート：**
```math
S^\dagger = S^3 = \begin{pmatrix} 1 & 0 \\ 0 & -i \end{pmatrix}
```
```math
T^\dagger = T^7 = \begin{pmatrix} 1 & 0 \\ 0 & e^{-i\pi/4} \end{pmatrix}
```

### 回転ゲートの合成

**同軸回転の合成：**
```math
R_a(\alpha)R_a(\beta) = R_a(\alpha + \beta) \quad (a = x, y, z)
```

**異軸回転（非可換）：**
```math
[R_x(\alpha), R_y(\beta)] \neq 0 \quad \text{（一般に）}
```


## 4. Pauliゲートの代数

### 反交換関係

```math
\{X,Y\} = XY + YX = 0
```
```math
\{Y,Z\} = YZ + ZY = 0  
```
```math
\{Z,X\} = ZX + XZ = 0
```

### 交換子

```math
[X,Y] = XY - YX = 2iZ
```
```math
[Y,Z] = YZ - ZY = 2iX
```
```math
[Z,X] = ZX - XZ = 2iY
```

### Pauli群の性質

```math
X^2 = Y^2 = Z^2 = I
```
```math
XYZ = iI
```

**他の順序の場合:**
```math
XZY = -iI
```
```math
YXZ = -iI  
```
```math
YZX = iI
```
```math
ZXY = iI
```
```math
ZYX = -iI
```

### 一般的な規則

**偶数順列（元の順序XYZからの偶数回の交換）:**
- XYZ → +i
- YZX → +i  
- ZXY → +i

**奇数順列（元の順序XYZからの奇数回の交換）:**
- XZY → -i
- YXZ → -i
- ZYX → -i



## 5. 任意ユニタリ分解

### ZXZ分解（オイラー角分解）

任意の1量子ビットユニタリは：
```math
U = e^{i\alpha}R_z(\beta)R_x(\gamma)R_z(\delta)
```

### ZYZ分解

```math
U = e^{i\alpha}R_z(\beta)R_y(\gamma)R_z(\delta)
```

### XYX分解

```math
U = e^{i\alpha}R_x(\theta_1)R_y(\phi)R_x(\theta_2)
```

これらの分解は一意に決まる（グローバル位相を除く）。


## 6. ゲート集合と普遍性

### Clifford群

**生成元：**
- Pauliゲート: `{X, Y, Z}`  
- Hadamardゲート: `H`
- 位相ゲート: `S`

**性質：**
- Pauliゲートを他のPauliゲート（符号込み）に共役変換
- 古典的に効率よくシミュレーション可能（Gottesman-Knill定理）

### 普遍ゲート集合

**Clifford + T:**
```math
\{H, S, T, \text{CNOT}\}
```

**理由：**
- Solovay-Kitaev定理により任意のユニタリを効率的に近似
- 耐故障量子計算で実用的
- Tは非Cliffordゲートで量子優位性を提供


## 7. CNOTとPauliの共役関係

### CNOTによる変換

```math
\text{CNOT} \cdot (X \otimes I) \cdot \text{CNOT}^\dagger = X \otimes X
```
```math
\text{CNOT} \cdot (I \otimes X) \cdot \text{CNOT}^\dagger = I \otimes X
```
```math
\text{CNOT} \cdot (Z \otimes I) \cdot \text{CNOT}^\dagger = Z \otimes I
```
```math
\text{CNOT} \cdot (I \otimes Z) \cdot \text{CNOT}^\dagger = Z \otimes Z
```

これらの関係は量子誤り訂正で重要。



## 8. 実用的な分解例

### CYゲート分解
```math
\text{CY} = (I \otimes S^\dagger) \cdot \text{CNOT} \cdot (I \otimes S)
```

### iSWAP分解
```math
\text{iSWAP} = \text{CNOT}_{12} \cdot (I \otimes S) \cdot \text{CNOT}_{12} \cdot (S \otimes I)
```

### 制御回転ゲート
```math
\text{CR}_z(\theta) = (I \otimes R_z(\theta/2)) \cdot \text{CNOT} \cdot (I \otimes R_z(-\theta/2)) \cdot \text{CNOT}
```


## 9. まとめ

自分の学習用のリファレンスとしてまとめたものなので、必要に応じて今後も追記・修正していきます。 もし間違いや不足があればフィードバックをいただき、さらに精度の高い「自分専用の量子ゲート図鑑」に育てていきたいと思います。  
