# カーネル関数による回帰

$n$個の入力$X$があるとします。

```math
X=\{X_1, X_2, ..., X_n\} \tag{1}
```

それぞれの$X$に対して観測値$Y$が得られたとします。

この所与のデータ$(X, Y)$について、関数$y = f(x)$を用いて近似する事を考えます。
$f(x)$は、$d$個の特徴量$φ(x)$の線型結合で得られるとしたら、

```math
f(x)=\sum_{m=1}^{d} α_m\ φ_m(x)\tag{2}\\
```

この近似式を、特殊な関数$k$を用いて書き直します。
関数$k$は、特徴量同士の内積として定義される。

```math
k(x_i, x_j):=φ(x_i)^Tφ(x_j)=\sum_{m=1}^dφ_m(x_i)φ_m(x_j)\tag{3}\\
```

<img src = md/復習のガウス過程/c0ce7645-4f51-f6cc-94a9-f54d0c44b29f.png width = 500>


特徴量マトリックス$φ(x)$をカラム方向に見て線型結合したのが(2)式なのだけど、関数$k$を用いることで、ロウ方向に見た線型結合に書き直すことができます。

```math
\begin{eqnarray}
f(x)&=&\sum_{m=1}^{d} α_m\ φ_m(x)\\
&=&\sum_{i=1}^n w_i\ φ(X_i)^Tφ(x)\\
&=&\sum_{i=1}^n w_i\ k(X_i, x) \tag{4}\\
\end{eqnarray}
```

この時、最小2乗誤差を与える$w$は、

```math
\begin{eqnarray}
\newcommand{\argmin}{\mathop{\rm arg~min}\limits}
&\argmin_{f(x)}&\ r_{square}&=&\sum_i^n \big(Y_i-f(X_i)\big)^2\\
\leftrightarrow&\ \argmin_{w}\  &r_{square}&=&\sum_{i=1}^{n}\big(Y_i - \sum_{j=1}^n w_i\ k(X_i,X_j)  \big)^2\\
\leftrightarrow & &w &=& (K+εI_n)^{-1}Y,\ K_{ij}=k(X_i,X_j) \tag{5}
\end{eqnarray}
```

で与えられます。$ε$は誤差項で$I_n$は$n\times n$の単位行列。

関数$k$が既知の場合は(5)式で済むけれど、一般には関数$k$を与えるハイパーパラメータ$θ$は未知であり、データからベイズ的に推定する必要がある。逆に言えば、関数$k$すなわちハイパーパラメータ$\theta$を推定する事さえできれば、(5)式を用いて(2)式の近似は達成される。

特徴量の数$d$は一般に未知の自然数で、やろうと思えば無限に与える事ができる。
しかし、データの数$n$は既知であり必ず有限である。言い換えれば、適切に設計された関数$k$を用いることで、有限の$n$の線形和を考えるだけで無限の$d$について回帰することが出来るという理屈です。

# カーネル関数の条件

さて、(3)式では、関数$k$を特徴ベクトル同士の内積として定義しました(再掲)。

```math
k(x_i, x_j):=φ(x_i)^Tφ(x_j)\tag{3'}\\
```

内積は、ベクトル同士が一致して入れば0、直行して入れば1を取るので、類似度とも言えます。関数$k$が類似度であるなら、類似度は関数$k$と考えることができるのか？結論を言えば、(条件を満たせば)できます。

条件は、任意の実数$c=c_1,c_2,...,c_n$に対し、(6)式を満たすこと。

```math
\sum_{i,j=1}^nc_ic_jk(x_i,x_j)\ >\ 0\tag{6}\\
```

左辺は、行列$K$の二次形式になっている。

```math
\mathbf{K}=\left( 
\begin{array}{}
k(x_1,x_1)&\ldots&k(x_1,x_n)\\
\vdots&\ddots&\vdots\\
k(x_n,x_1)&\ldots&k(x_n,x_n)\\
\end{array}
\right) \tag{7}
```

# ガウス過程

ところで、観測誤差εを正規分布$\mathcal{N}$で与えて下記の様に書くことがあります。

```math
\begin{eqnarray}
&Y&=&f(x)+\mathcal{N}(0,ε)\\
\leftrightarrow\ &Y&\sim&\mathcal{N}\big(f(x),ε\big)\tag{8}
\end{eqnarray}
```

$f(x)$の周りに正規誤差がくっついているというのが直感的な理解。
ここで視点を変えてみる。

<img width="700" alt="スクリーンショット 2018-08-20 12.59.42.png" src="md/復習のガウス過程/009eb432-8251-7156-4308-ad868cf2597f.png">

$n$次元正規分布から、出力$Y=[Y_1, Y_2, ..., Y_n]$をサンプルしてくる、と考える。

```math
\begin{eqnarray}
&Y&\sim&\mathcal{N_n}(\mu,\Sigma)\tag{9}
\end{eqnarray}
```

$\mu$は期待値で、$\Sigma$は偏差の積の期待値($n \times n$行列)。


```math
\begin{eqnarray}
\mu&=&\mathbf{E}[f(x)]\tag{10}\\
\Sigma_{ij}&=&\mathbf{E}\Big[\big(f(x_i)-\mu(x_i)\big)\ \big(f(x_j)-\mu(x_j)\big)\Big]\tag{11}
\end{eqnarray}
```

で、(11)式の$\Sigma$の要素は(6)式の性質を満たすので、関数$k$として扱うことができます。
この時、$\Sigma$は関数$k$に対する行列$K$に相当します。

```math
\Sigma_{ij}=k(x_i,x_j)=\mathbf{E}\Big[\big(f(x_i)-\mu(x_i)\big)\ \big(f(x_j)-\mu(x_j)\big)\Big]\tag{12}
```

(10)式と(12)式で与えられる(9)式で定義される関数$f(x)$のサンプリングをガウス過程と呼びます。

```math
f(x)\sim\mathcal{GP}\big(\mu,\ k\big) \tag{13}
```

実用上は、出力$Y$をスケールしてしまい、$\mu=\vec{0}$として$\Sigma$のみを考えるので十分。また、よく使われる関数$k$は、3つのハイパーパラメータ$\theta=(\eta,\rho,\sigma)$を用いて、下記の形で仮定します。($\delta$はクロネッカのデルタ)

```math
\begin{eqnarray}
&Y&\sim&\mathcal{N_n}(\vec{0},\mathbf{K})\\
&\mathbf{K}_{ij}&=&k(x_i,x_j)\\
&&=&\eta^2exp\big(-\rho^2(x_i-x_j)^2\big)+σ^2δ_{ij}\tag{14}
\end{eqnarray}
```

ふ〜、シンプルですね。
入力$X$・出力$Y$が与えられた時、このモデルに従ってハイパーパラメータ$\theta$を推定できる。

# ガウス過程を用いた回帰

上記の議論は色々なところをボヤかして書かれています。
例えば図中の空間Aでは、$x$軸上に無限の点を取ることができます。
従って、$f(x)$を完全に記述するには、空間Bは無限次元になるはずです。

逆にいえば、実データ$(X, Y)$で与えられる$n$次元正規分布は、無限次元正規分布を少数の次元で近似したもの、という事になります。

ところで回帰って何かというと、新たな$X_{new}$に対応する測定していない$Y_{new}$を推定することです。
これは同一の無限次元正規分布を仮定して、測定部分$n$次元に基づいて否測定部分を推定する課題、と言い換えることができます。

つまり、$Y_1$と$Y_2$を同一の$Y$から派生した互いに素なデータとして、

```math
\begin{bmatrix}Y_1 \\ Y_2\end{bmatrix} \sim \mathcal{N_n}\begin{pmatrix}
\mu,
\Sigma\end{pmatrix} \tag{15}
```

この時、$\Sigma$は、適切な行列$A$を持ちいて、

```math
\Sigma = \begin{bmatrix} \Sigma_1 & A\\A^T & \Sigma_2
\end{bmatrix}\tag{16}
```

前節の議論から、既知の$Y_1$について下記のように置き、$k$を与えるハイパーパラメータ$θ$を推定可能。

```math
\begin{eqnarray}
&Y_1&\sim&\mathcal{N}_{n1}(\mu_1,\Sigma_1) \\
&\Sigma_{1ij}&=&k(X_{1i},X_{1j}|\theta)\tag{17}
\end{eqnarray}
```

元の分布$\mathcal{N}_n$が同一なので、$\theta$も共通のはず。
推定された$θ$を元に、$\Sigma$の他の要素を計算することが出来る。


```math
\begin{eqnarray}
\Sigma_{2ij}&=&k(X_{2i},X_{2j}|θ)\\
A_{ij}&=&k(X_{1i},X_{2j}|θ)\tag{18}
\end{eqnarray}
```

で、知りたいのは、条件付き分布。

```math
Y_{2|1}\sim\mathcal{N}_{n2}(\mu_{2|1},\Sigma_{2|1})\tag{19}
```

ここで、$\mu=\vec{0}$として、多変量正規分布の性質から、

```math
\begin{eqnarray}
\mu_{2|1}&=& A^t\Sigma_1^{-1}Y_1\\
\Sigma_{2|1}&=&\Sigma_2-A^t \Sigma_1^{-1} \tag{20}
\end{eqnarray}
```

こうして(19)式の右辺が求まるので、その多変量正規分布からサンプリングを実行出来る。
結果、既知の入力$X_1$と出力$Y_2$から、任意の新たな入力$X_2$に対する出力$Y_{2}$を推定することが可能になる。

----

既存の記事を書き換えようとするとカオスになるので、何度も同じ様な記事を書く羽目になっています。しかしその都度、自分の中ではモヤモヤした点があり、記事をまとめる過程でクリアになっていくという状況です。恐らく、ガウス過程の理論部分に関してはこれで終わりになるかなと思っています。Stan manualの和訳が残っているのでそちらも順次片付けていくつもりです。


----
# 教科書
・カーネル多変量解析 [Amazon](https://www.amazon.co.jp/dp/4000069713/)
・GPML [link](http://www.gaussianprocess.org/gpml/)
・カーネル法 -正定値カーネルを用いたデータ解析 [link](http://www.ism.ac.jp/~fukumizu/ISM_lecture_2004/Lecture2004_kernel_method.pdf)

過去記事
・[ガウス過程の考え方](https://qiita.com/kilometer/items/6ab1d7f2cffc0c6e4a95)
・[{rstan} Rstanでガウス過程の実装](https://qiita.com/kilometer/items/8b81560c0efef5e0cee2)
・[オレオレStan manual和訳：18章 ガウス過程](https://qiita.com/kilometer/items/91f44b28f1c2ea82bc73)
・[R de 『カーネル多変量解析』](https://qiita.com/kilometer/items/58376b9a103743329b2f)
・[カーネルトリック](https://qiita.com/kilometer/items/66e6116cc661019ead59)
・[条件付き多変量正規分布](https://qiita.com/kilometer/items/34249479dc2ac3af5706)

