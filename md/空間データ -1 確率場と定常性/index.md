# 0. 確率場
## 0-1. 確率場
同時分布が確率的に与えられる有限個の実数からなる集合を**確率場**と定義し、
$Y(s): s∈D_S⊂R^d$
と表記する。

観測可能なのは、これに測定誤差$ε$を加えた実測値$Z(s)$のみである。
$Z(s) = Y(s) + ε(s)$

$Z(s_i); s∈D_s$から測定誤差と真値の空間構造をモデル化する事で、
非観測地点における真の値$Y(s_0)$を推定する。

実用的には、例えば、空間補完などに用いられる。
ただし、時間構造を含まない**temporary frozen**なデータ構造を対象にしている事に留意。


階層的地統計モデルでは、2つのモデルを考える。

```math
\begin{eqnarray}
&data\ model &&\\
& && Z(s) = Y(s)+ε(s),\hspace{10pt} idd.\ & ε\ \thicksim \ N(0, σ^2_ε) \\
 \\
&process\ model &&\\
& && Y(s) = X(s)^t β + σ(s),\ & σ\ \thicksim \ N(0, C_Y(h)), \\
&&& C_Y(h)=cov\left(Y(s+h), Y(s)\right)
\end{eqnarray}
```
すなわち、process modelにおける(多変量)正規分布誤差が、測定地点の相互位置に依存して変動する真の値$Y$の共分散$C_Y(h)$で与えられると考える。$X$は説明変数行列。



## 0-2. 定常性
解析的には、 **定常性**を仮定する。
定常性とは、与えられた$k$個の点の組が**互いの相対的位置関係(空間データでは、距離, 方位, 高度, ...etc)を保ったまま他の部分に移動しても特徴が不変である**という事を意味し、場の1次モーメント, 2次モーメントにより定義される。

# 1. 固有定常性 *intrinsic stationary*

```math
\begin{eqnarray}
^{\forall}s,^{\forall}h ∈ D_s,&&\hspace{130pt} \\

&{\bf E}\left[Y(s+h)-Y(s)\right]=0,\hspace{30pt}   &\cdots(1.1)\\
&var\left(Y(s+h)-Y(s)\right)=2γ_Y(h)\hspace{10pt} &\cdots(1.2)

\end{eqnarray}
```

$2γ_y(h)$を**バリオグラム variogram**、$γ_y(h)$を**セミバリオグラム**と呼ぶ。

(1.3)式を満たさないバリオグラムは不適。
これを`conditional-nonpositive-definiteness condition`と呼ぶ。

```math
^{\forall} k ∈ {\bf N}, ^{\forall}\{s_i: i=1,...k\},
^{\forall}α_i\ such\ that\sum_{i=1}^kα_i=0, \hspace{130pt} \\

\sum_{i=1}^{k} \sum_{j=1}^{k} α_i α_j 2γ_Y \left(s_i - s_j\right)≤0 
\hspace{10pt} \cdots(1.3)
```

##  数学的意味
$\left(\sum_{i=1}^kα_iY(s_i)\right)^2$の期待値を考える。
まず、
$var(A)={\bf E}[A^2]+\left({\bf E}[A]\right)^2$を用いて、

```math
{\bf E}\Biggl[\left(\sum_{i=1}^k α_i Y(s_i)\right)^2\Biggr] \hspace{150pt} \\
\begin{eqnarray}
&=& var \left(\sum_{i=1}^k α_i Y(s_i) \right)
 + \biggl({\bf E} \left[\sum_{i=1}^k α_i Y(s_i) \right]\biggr)^2 \\
&=& var \left(\sum_{i=1}^k α_i Y(s_i) \right) 
+ \biggl(\sum_{i=1}^k α_i\ {\bf E}\left[Y(s_i) \right]\biggr)^2 \\
&=& var \left(\sum_{i=1}^k α_i Y(s_i) \right) \hspace{80pt}
\cdots(1.4) \\
&& \hspace{100pt}  ∵\ \sum_{i=1}^k α_i=0
\end{eqnarray}
```
また、

```math
\begin{eqnarray}
\left(\sum_{i=1}^kα_iY(s_i)\right)^2\\
&=&\sum_{i=1}^k\sum_{j=1}α_i α_j Y(s_i) Y(s_j) \hspace{110pt} \\
&=&\sum_{i=1}^k\sum_{j=1}\frac{α_i α_j}{-2}\biggl(\left(Y(s_i)-Y(s_j)\right)^2-Y(s_i)^2-Y(s_j)^2\biggr) \\
&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \left(Y(s_i)-Y(s_j)\right)^2 \\
&& +\frac{1}{2}\sum_{i=1}^k α_i Y(s_i)^2 \sum_{j=1}^kα_j 
+\frac{1}{2}\sum_{i=1}^k α_i \sum_{j=1}^k α_j Y(s_j)^2  \\
&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \left(Y(s_i)-Y(s_j)\right)^2 \\
\end{eqnarray}
```
従って、(1.1)(1.2)を用い、

```math
\begin{eqnarray}
{\bf E}\Biggl[\left(\sum_{i=1}^k α_i Y(s_i)\right)^2\Biggr] \\
&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \ 
{\bf E} \biggl[\left(Y(s_i)-Y(s_j)\right)^2 \biggr] \hspace{80pt}\\
&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \ 
\Biggl(var\left(Y(s_i)-Y(s_j)\right)
+\biggl[{\bf E}\left(Y(s_i)-Y(s_j)\right)\biggr]^2 \Biggl) \\

&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \ 
var\left(Y(s_i)-Y(s_j)\right) \\
&=&-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \ 2γ_Y(s_i-s_j)
\hspace{40pt}
\cdots(1.5)

\end{eqnarray}

```

(1.4), (1.5)より、

```math
 var \left(\sum_{i=1}^k α_i Y(s_i) \right) 
=-\frac{1}{2}\sum_{i=1}^k\sum_{j=1}^k α_i α_j \ 2γ_Y(s_i-s_j)
```
すなわち、右辺の$var(\ )$が非負($≥0$)である事が、(1.3)式と同値。


# 2. 2次定常性 *2nd order stationary*

```math
\begin{eqnarray}
^{\forall}s,^{\forall}h ∈ D_s,&&\hspace{130pt} \\

&{\bf E}\left[Y(s)\right]=μ,\hspace{30pt}   &
\cdots(2.1)\\
&cov\left(Y(s+h), Y(s)\right)=C_Y(h)\hspace{10pt} &
\cdots(2.2)

\end{eqnarray}
```
この時、$C_Y(h)$を共分散関数、もしくは、コバリオグラムcovariogramと呼ぶ。
"定常的"コバリオグラムは、下記を満たす(満たさない場合、不適)。

```math
\begin{eqnarray}
^{\forall} α_i∈R, ^{\forall} S_i∈D_s, &\\
\sum_{i=1}^k \sum_{j=1}^k  α_i α_j\ & C_Y(s_i-s_j)\ ≥ 0 \hspace{10pt} \cdots(2.3)\\
\end{eqnarray}
```
※ ここで、$\sum α_i=0$ の縛りが必要無いことに注意。
つまり、2次定常性の方が、固有定常性よりも**弱い**仮定。
(2次定常性を弱定常性weak stationaryとも呼ぶ)

2次定常性ならば、固有定常性で、かつ、下記が成立する。

```math
γ_Y(h)=C_Y(0)\ -\ C_Y(h) \hspace{30pt} \cdots(2.4)
```
証明は下記。




## 数学的意味
式(2.3)これはほとんど自明ですが、

```math
\begin{eqnarray}
var \left( \sum_{i=1}^k α_i Y_s \right) && \\
&=& \sum_{i=1}^k \sum_{j=1}^k α_i α_j\ cov(Y(s_i), Y(s_j)) \\
&=& \sum_{i=1}^k \sum_{j=1}^k α_i α_j\ C_Y(s_i -s_j)  \hspace{5pt}≥0\\
\end{eqnarray}
```

式(2.4)の証明は、下記。
式(1.2)および、$var(aX+bY)=a^2\ var(X)+b^2\ var(Y)-2ab\ cov(X,Y)$を用い、

```math
\begin{eqnarray}
γ_Y(h)&& \hspace{150pt} \\
&=&\frac{1}{2} var \left( Y(s+h)-Y(s) \right) \hspace{150pt}\\
&=&\frac{1}{2} \left(var(Y(s+h)+var(Y(s)-2cov(Y(s+h), Y(s))\right) \\
&=&\frac{1}{2} \left(C_Y(0) + C_Y(0) -2C_Y(h) \right) \\
&=& C_Y(0)-C_Y(h)
\end{eqnarray}
```


