# 2. Simple Kriging
## 2-0. 基本的なアイディア

前記事より、
[空間データ-1 確率場と定常性](http://qiita.com/kilometer/items/553b344c818566dbb37d)
　　　0. 確率場
　　　1. 固有定常性
　　　2. 2次定常性

端的には、線形モデルによる内挿(空間補間)を目的とする。

地点$s$における空間データ$Z(s)$の集合は、確率場$Y(s)$に測定誤差$ε$を加え、
$Z(s)=Y(s)+ε(s)$
として考える。

$Z(s_i); s∈D_s$から測定誤差と真値の空間構造をモデル化する事で、
非観測地点における真の値$Y(s_0)$を推定する。


Krigingでは、$Y(s)$に対して**2次定常性**を仮定する。
更に、空間変動を記述する$Y(s)-Y(s+h)$に対し、**固有定常性**を仮定する。

***
**2次定常性**
　任意の2点の組が、互いの相対的位置関係を保ったまま、他の場所に移動しても、特徴が不変である、すなわち、期待値と共分散関数が共に移動不変である性質

***
**固有定常性**
任意の対をなす2点の確率場の差分に対して、1次および2次モーメントが不変である性質

***

## 2-1. Simple Kriging


確率場$Y(s): s∈D_S⊂{\bf R}^d$について、

```math
\begin{eqnarray}
{\bf E}[Y(s)]&=&μ_Y(s) \hspace{70pt} &\cdots(1)\\
C_Y(μ, v)&=&cov\left(Y(μ), Y(v)\right)&\cdots(2)\\
mesurement\ error\ &=&\ σ^2_ε&\cdots(3)\\
\end{eqnarray}
```

を、**既知**とする。
ベイズ的には一度に推定したい所。

$Y(s_0)$の回帰式は、

```math
Y(s_0) = l(s_0)^t \ Z+k(s_0) \hspace{30pt} \cdots(4)
```
$l∈{\bf R}^m$は場所依存的なウェイト項。$k(s_0)∈{\bf R}^m$は撹乱項。
となると、最小二乗誤差(Mean Squared prediction error, MSPE)は、

```math
\begin{eqnarray}
MSPE(l,k) &=& {\bf E}\left[Y(s_0)-l^t \ Z-k\right]^2 \\
&=& var \left(Y(s_0)-l^t \ Z-k \right)+\biggl({\bf E}\left[Y(s_0)-l^t\ Z-k)\right]\biggr)^2 \hspace{10pt} \cdots(5)
\end{eqnarray}
```

(5)式第2項が最小になるのは、第2項=0として、

```math
\begin{eqnarray}
k^*
&=& {\bf E}\left[Y(s_0)\right]-l^t\ {\bf E}\left[Z\right] \\
&=& μ_Y(s_0)-l^t\ {\bf E}[Y] \\
&=& μ_Y(s_0)-l^t\ μ_Y(s) \hspace{30pt} \cdots(6)
\end{eqnarray}
```
(6)を5式に代入し、$μ_Y$は定数である事に注意して、

```math
\begin{eqnarray}
MSPE(l,k=k^*) &=& var \left(Y(s_0)-l^t \ Z-k^* \right) \\
&=& var \left(Y(s_0)-l^t \ Z -μ_Y(s_0)+  l^t\ μ_Y(s) \right) \\
&=& var \left(Y(s_0) -μ_Y(s_0) -l^t(Z-μ_Y(s)) \right) \\
&=& var\left(Y(s_0)-μ_Y(s_0)\right)-var\left(l^t(Z-μ_Y(s)) \right)\\
&& \hspace{5pt}-2\ cov \left(Y(s_0)-μ_Y(s_0), l^t(Z+μ_Y(s))\right) \\
&=& var \left(Y(s_0) \right)+var(l^t\ Z)-2l^t\ cov(Y(s_0), Z)\\
&=& cov(Y(s_0), Y(s_0))+l^t\ cov(Z(s_i), Z(s_i))\ l-2l^tcov(Y(s_0), Z)\\
&=& C_Y(s_0)+l^t\ C_Z\ l-2l^t cov(Y(s_0), Z) \hspace{50pt} \cdots(7)\\

\end{eqnarray}
```

(7)を、$l$について偏微分する。

```math
\begin{eqnarray}
\frac{\partial{MSPE(l,k^*)}}{\partial{l}}
&=&\frac{1}{\partial{l}}\biggl( -2cov(Y(s_0), Z) + C_Z\ l\biggr) \\
\Leftrightarrow \ 0 &=&-2cov(Y(s_0)^*, Z) + C_Z\ l^* \\
\Leftrightarrow \ l^* &=& C_Z^{-1}cov(Y(s_0)^*, Z) \hspace{10pt} &\cdots(8)

\end{eqnarray}
```
(6)(8)を(4)に代入し、

```math
\begin{eqnarray}
Y(s_0)^* &=& l^{*\ t} \ Z+k^* \\
&=& l^{*\ t}\ Z +  μ_Y(s_0)-l^{*\ t}\ μ_Y(s) \\
&=& μ_Y(s_0) + l^{*\ t} \left(Z-μ_Y(s) \right) \\
&=& μ_Y(s_0) + \biggl( C_Z^{-1}cov(Y(s_0)^*, Z) \biggr)^t \left(Z-μ_Y(s) \right) \\
&=& μ_Y(s_0) + C_Z^{-1}\ cov \left(Y(s_0)^*, Z \right)^t\left(Z-μ_Y(s) \right) 
\hspace{30pt} \cdots(9)
\end{eqnarray}
```

これがsimple krigingの予測子になる。

そして、最小二乗誤差は、(simple) kriging varianceと呼ばれ、下記で与えられる。

```math
\begin{eqnarray}
σ^2_{Y}
&=& MSPE(l^*, k^*) \\
&=& C_Y(s_0) - l^{*\ t} \left(C_Z\ l^*-2cov(Y(s_0)^*,Z) \right) \\
&=& C_Y(s_0) - l^{*\ t} \left(C_Z\ C_Z^{-1}cov(Y(s_0)^*, Z) -2cov(Y(s_0)^*,Z) \right) \\
&=& C_Y(s_0) + l^{*\ t} cov(Y(s_0)^*, Z) \\
&=& C_Y(s_0) + cov(Y(s_0)^*,Z)^t\ C_Z^{-1}\ cov(Y(s_0)^*,Z)
\hspace{30pt} \cdots(10)
\end{eqnarray}
```


