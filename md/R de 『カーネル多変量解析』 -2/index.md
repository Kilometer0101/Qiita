引き続き[『カーネル多変量解析』](https://www.amazon.co.jp/dp/4000069713) 赤穂昭太郎 (岩波書店)

承前、[R de 『カーネル多変量解析』](http://qiita.com/kilometer/items/58376b9a103743329b2f)

大文字の`X`と小文字の`x`が出てきますが、大文字は入力データを意味し、
以下の議論において所与とする。

```math
X=(X_1,\ X_2,\ ...,\ X_n)
```


# 復習

先だっての第1章では、カーネル関数を用いた回帰の例を挙げていました。

線形回帰と同様、データ`(X,Y)`に対して`y=f(x)`で近似しますが、
カーネル関数を使った近似では、この`f(x)`が特殊な形をしている。
回帰係数を`w_1~n`として、

```math
f(x)=\sum_{i=1}^n w_i\ k(X_i, x) \tag{0}
```

# 特徴量

2章ではまず、**特徴量**という概念が説明されています。

多項式近似のイメージを使って説明してみましょう。
<img width=300 src=md/R de 『カーネル多変量解析』 -2/31ae6136-ef6e-3468-3ea4-07aad0dab736.png>

```R
set.seed(111)
N <- 100
x <- runif(N, -2, 2)
y <- 2*x^2+3*x+rnorm(N, 0, 2)
```

こんなデータ`(X, Y)`を考えます。
入力は**一次元**の変数`X`で、観測量が`Y`。

`X`の一次式で表せる関数(線形)では、直線的なカンケイしか表せないので、
曲線の式`y=f(x)`には、次数`d`の多項式を考える。

```math
f(x)=\sum_{m=1}^d α_m x^m
```

この多項式は、`x`の線形ではないですが、
推定すべき**αについて**線形になっている事に注意しておきます。

サンプルデータでは2次式`(d=2)`と考えて、下記を用いる。

```math
y=α_1*x+α_2*x^2
```

そんな事は知っているよ！だから何だよ！（ーー；）

ええ。だから、ですね。

```R
# 必要に応じて、事前に install.packages("rgl")
library("rgl")

x2 <- x^2
plot3d(cbind(x, x2, y), col="Red", size=6)
```
<img width=450 src=md/R de 『カーネル多変量解析』 -2/43a2b8ce-e3bb-87a7-b268-9f8875e4d3eb.png>

これをグリグリ回すと分かると思います。
「xとyのカンケイ」は、単純な線や面になっていませんので、面倒。
「xとx²とyのカンケイ」にすると、明らかに平面に乗っている。
線や平面や超平面は、線形和で書けるはずなので簡単に解ける。

1次元の入力`X`から2次元の特徴量`x, x²`を抽出した空間でデータを評価する事で、より高精度な近似が(単純な線形回帰により)得られる。

<img width=300 src=md/R de 『カーネル多変量解析』 -2/7d32e5bb-7e3e-98dd-0558-5cc01ef1e9b4.png>

```R
x_k <- seq(-2,2, 0.1)

y_1 <- predict(lm(y~x), data.frame(x=x_k))
y_2 <- predict(lm(y~x2+x), data.frame(x=x_k, x2=x_k^2))


par(tcl=-0.2, mgp=c(1.5, 0.3, 0))
plot(x,y, pch=16, col="Red")
 lines(x_k, y_1)
 lines(x_k, y_2)
```
しつこいですが、d個の特徴量のベクトルを`φ`と表すと、

```math
\begin{eqnarray}
φ(x)&=&\big(φ_1(x)=x,\ φ_2(x)=x^2, ...,\ φ_d(x)=x^d \big) \\
α&=&\big(α_1,\ α_2,\ ...,α_d\big)
\end{eqnarray}
```
の元、

```math
\begin{eqnarray}
f(x)&=&\sum_{m=1}^d α_m x^m=\sum_{m=1}^d\ α_m\ φ_m(x) \\
&=&α^Tφ(x)
 \tag{1}
\end{eqnarray}
```

という事です(`x`の線形ではなく`φ`の線形)。

# 特徴ベクトルからみたカーネル関数

データ集合`X`に対し、カーネル関数は、含まれる2点`X_i`, `X_j`における**特徴ベクトル同士の内積**として定義できる。

上の例では、2つの特徴ベクトル`x, x²`を用いているので、
![スクリーンショット 2016-10-29 16.22.06.png](md/R de 『カーネル多変量解析』 -2/a7d46b78-b189-aec6-33de-b2914698270b.png)

```math
\begin{eqnarray}
k(X_i, X_j)&=&φ(X_i)^Tφ(X_j) \\
&=& \big( φ_1(X_i),\ φ_2(X_i) \big)^T\big( φ_1(X_j),\ φ_2(X_j) \big) \\
&=& X_iX_j+X^2_iX^2_j 
\end{eqnarray}
```


Rで内積マトリックス`K`を書いてみると、

```math
K_{ij}=k(X_i, X_j)
```

<img width=350 src=md/R de 『カーネル多変量解析』 -2/6ef3b501-fe24-d409-8350-13a34b14aae5.png>

```R
dat <- data.frame(x, x2)

K <- matrix(0, N, N)
for(i in 1:N){
	for(j in 1:N){
		K[i, j] <- sum(dat[i,]*dat[j,])
}}
image(1:N, 1:N, K, col=grey(1:100/100))
```
<img width=400 src=md/R de 『カーネル多変量解析』 -2/28b7d77d-2132-888a-e772-69728c8c89bb.png>
当然、対称行列ですね。
この行列は、**半正定値性**を満たすという重要な性質がありますが、ここでは踏み込まない。

さて、任意の`x`における入力`X`に対するカーネル関数の積和は、

```math
f(x)=\sum_{i=1}^n k(X_i,x)
```

```r
y_k <- rep(0, length(x_k))
for(k in 1:length(x_k)){
	for(i in 1:N){
		y_k[k] <- y_k[k]+x[i]*x_k[k]+x[i]^2*x_k[k]^2
}
}
```
<img width=350 src=md/R de 『カーネル多変量解析』 -2/66d71b39-6e6e-d186-e451-53709c4a0b30.png>

適切な重み`w_i`を用いる事で、データ`(X,Y)`を近似できそうだ。

```math
\begin{eqnarray}
f(x)&=&\sum_{i=1}^n w_i\ k(X_i, x) \tag{2}\\
&=&\sum_iw_i\ φ(X_i)^T\ φ(x)\\
&=&\Big(\sum_iw_i\ φ(X_i)\Big)^T\ φ(x) \tag{3}\\
\end{eqnarray}
```

(0)と(2)が同じ形をしている事に注意。
実は、これは自明ではなく、証明が必要であるが、ひとまず割愛。

(1)(3)を見比べて、

```math
\begin{eqnarray}
α&=&\sum_{i=1}^n w_i\ φ(X_i) \\
α_m&=&\sum_{i=1}^n w_i\ φ_m(X_i) \\
&=&\sum_{i=1}^n w_i\ X_i^m &(m=1,2,...,d)
\end{eqnarray}
```
と置く事で、

```math
f(x)=\sum_{m=1}^d α_m x^m=\sum_{i=1}^n w_i\ k(X_i, x) \tag{4}\\
```

すなわち、**多項式近似において次数`d`個の係数`α`を推定することは、
カーネル関数を用いてデータ数`n`個の係数`w`を推定する事と表せる**。


# 実装
[前回の議論](http://qiita.com/kilometer/items/58376b9a103743329b2f)は、今回のカーネル関数においてもまったく同様なので、(4)を用いた最小二乗近似を満たす`w`は、

```math
w=K^{-1}Y
```

で求まるはず。

例によって`solve`の問題があるので、逆行列は`MASS::ginv`を用いてムーアペンローズの疑似逆行列で代用。

```R
library(MASS)
set.seed(111)

N <- 100
x <- runif(N, -2, 2)
x2 <- x^2
y <- 2*x^2+3*x+rnorm(N, 0, 2)

dat <- data.frame(x, x2)

K <- matrix(0, N, N)
for(i in 1:N){
	for(j in 1:N){
		K[i, j] <- sum(dat[i,]*dat[j,])
}}

w <- ginv(K) %*% y

x_k <- seq(-2, 2, 0.1)
y_k <- rep(0, length(x_k))
for(k in 1:length(x_k)){
	for(i in 1:N){
		y_k[k] <- y_k[k]+w[i]*(x[i]*x_k[k]+x[i]^2*x_k[k]^2)
		}}

par(tcl=-0.2, mgp=c(1.5, 0.3, 0))
plot(x,y, col="Red", pch=16)
lines(x_k, y_k, lwd=5)
lines(x_k, y_2, lty=2, lwd=3, col="green") # 大昔に作った多項式近似
```

<img width=450 src=md/R de 『カーネル多変量解析』 -2/a3d2fd47-5ed5-b2b2-94a7-62d1a8d3cd20.png>

カーネル近似が黒、多項式が緑破線。


ふ〜。
for文の嵐。



enjoy!!

