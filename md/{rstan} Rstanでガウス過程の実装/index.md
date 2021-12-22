一次元のN個の入力$x$と対応する出力$y$を想定する。
出力が未知の入力$x_2$を考え、これに対応する$y_2$を推定する。

承前：[ガウス過程の考え方](http://qiita.com/kilometer/items/6ab1d7f2cffc0c6e4a95)

参考：
・[ガウス過程シリーズ1 概要](http://statmodeling.hatenablog.com/entry/gaussian-process-1)
・[ガウス過程シリーズ2 高速化&フルベイズ](http://statmodeling.hatenablog.com/entry/gaussian-process-2)
・[『StanとRでベイズ統計モデリング』](http://www.kyoritsu-pub.co.jp/bookdetail/9784320112421)
・[『Gaussian Processes for Machine Learning』](http://www.gaussianprocess.org/gpml/)
・[『パターン認識と機械学習 (下)』](https://www.amazon.co.jp/dp/4621061240)

# 0. sample data
まずは**R**でサンプルデータを作ろう。

```math
y \sim N(sin(x),\ ε)
```

で、いっか。

```r
set.seed(123)
N <- 20
x <- runif(N, -5, 5)
y <- sin(x) + rnorm(N, 0, 0.01)
```

`x2`も準備しておく。

```r
N2 <- 75
x2 <- seq(-5, 5, length= N2)
```


# 1. stan code
ではstan。

以下、ブロックごとに順番に書いていけば良い。

## 1-1. data
上3行は既知のxとそれに対応するyの入力。
下2行は、予測したい定義域x2を格納。

```stan
data{
 int<lower = 1> N;
 vector[N] x;
 vector[N] y; 

 int<lower = 1> N2;
 vector[N2] x2;
}
```
## 1-2. transformed data

```math
y\sim N_n(\mu,\ cov)\tag{1}
```

としたい(コード中で $\mu$ は`mu`)。
データyを事前に標準化する事で、$\mu=0$と仮定する。
$\mu$の長さは当然$y$と等しいのでN。

```stan
transformed data{
 vector[N] mu;
 for(i in 1:N)
   mu[i] = 0;
}
```

## 1-3. parameters
式(1)の$cov$を推定したい。
kernel関数を用い、

```math
cov[i, j] = k(x_i,x_j)
=\eta^2exp\big(-\rho^2(x_i-x_j)^2\big)+σ^2δ_{ij} \tag{2}
```
($\delta_{ij}$はクロネッカのデルタ)

従って、推定すべきハイパーパラメータは3つ。
$\eta^2$(`eta_sq`), $\rho^2$(`rho_sq`), $\sigma^2$(`sigma_sq`)。 

```stan
parameters{
 real<lower = 0> eta_sq;
 real<lower = 0> rho_sq;
 real<lower = 0> sigma_sq;
}
```

## 1-4. transformed parameters
式(2)をそのまま書く。
$cov$は`Cov`。

```stan
transformed parameters{
 matrix[N, N] Cov;

 for(i in 1:(N-1)){
   for(j in (i+1):N){
     Cov[i, j] = eta_sq * exp(- rho_sq * pow(x[i] - x[j], 2));
     Cov[j, i] = Cov[i, j];
   }
 }

 for(k in 1:N)
   Cov[k, k] = sigma_sq;
}
```

## 1-5. model
モデルは式(1)そのもの。
$Cauchy$分布を使って裾野広い事前分布を与えておく。

```stan
model{
 y ~ multi_normal(mu, Cov);

 eta_sq ~ cauchy(0, 5);
 rho_sq ~ cauchy(0, 5);
 sigma_sq ~ cauchy(0, 5); 
}
```

## 1-6. generated quantities
で、$y$の推定結果に基づいて、いよいよ、$y_2$の推定。
式(1)で与えられる$y$の分布による条件付き多変量正規分布を推定する。
$\mu_2$(`mu2`)と$cov_2$(`Cov2`)が分かれば、$y_2$(`y2`)が推定できる。

```math
y_2 \sim N_{n_2}(\mu_2,\ cov_2)\tag{3}
```
$y$と$y_2$が同一の多変量正規分布から派生したとして、

```math
\begin{bmatrix}y \\y_2 \end{bmatrix} \sim N_{n+n_2}(\hat{\mu},\ \hat{cov})\tag{4}
```

標準化を前提として、$\hat{\mu}=0$。

```math
\hat{cov} = \begin{bmatrix}cov & K \\ K^t & \Sigma\end{bmatrix} \tag{5}
```

この時、$K$(`K`)は、

```math
K(x_{i}, x_{2j})=\eta^2exp\big(-\rho^2(x_{i}-x_{2j})^2\big) \tag{6}
```
$x$と$x_2$の長さが違う事に注意。(`K`は正方行列では無い。)

また、$\Sigma$(`Sigma`)は、

```math
\Sigma(x_{2i}, x_{2j})=\eta^2exp\big(-\rho^2(x_{2i}-x_{2j})^2\big)+σ^2δ_{ij} \tag{7}
```



求めたい$\mu_2$(`mu2`)と$cov_2$(`Cov2`)は、条件付き多変量正規分布の性質から、

```math
\begin{eqnarray}
\mu_2&=& 0+K^tcov^{-1}(y-\mu)=K^tcov^{-1}y\\
cov_2&=&\Sigma-K^t cov^{-1} \tag{8}
\end{eqnarray}
```

$\mu_2$の初項のゼロは何かというと、$\hat{\mu}=(\mu, \hat{\mu_2})$の$\hat{\mu_2}=0$。
記号がややこしいのであえて書かなかった。詳細は[前回記事](http://qiita.com/kilometer/items/6ab1d7f2cffc0c6e4a95)。

これを実装。
stanでは、$K^t$は、`K'`と書くことに注意して、書き下す。

```stan
generated quantities{
  vector[N2] y2;
  vector[N2] mu2;
  matrix[N2, N2] Cov2;
  matrix[N, N2] K;
  matrix[N2, N2] Sigma;
  matrix[N2, N] K_t_Cov;

// 式(6)
  for (i in 1:N)
    for (j in 1:N2)
      K[i, j] = eta_sq * exp(-rho_sq * pow(x[i] - x2[j],2));

// 式(7)
  for(i in 1:(N2-1)){
    for(j in (i+1):N2){
      Sigma[i, j] = eta_sq * exp(-rho_sq * pow(x2[i] - x2[j], 2));
      Sigma[j, i] = Sigma[i,j];
    }
  }
  
  for(k in 1:N2)
    Sigma[k, k] = sigma_sq;

// 式(8)-1
  K_t_Cov = K' / Cov;  
  mu2 = K_t_Cov * y;

// 式(8)-2
  Cov2 = Sigma - K_t_Cov * K;

  for(i in 1:N2)
    for(j in (i+1):N2)
      Cov2[i, j] = Cov2[j, i];

// 式(3)
  y2 = multi_normal_rng(mu2, Cov2);
}
```

※ 最後の式(3)の実装部分。
stanコード中の$\sim$は、sampling statement formという。
内部では対数確率の足し上げをしている。

今回は乱数を生成したいので、`multi_normal_rng`を使う。
`multi_normal_rng`をコレスキー分解を使って回避して、`normal_rng`のみで計算する事も可能です。
cf. [続ガウス過程実装：コレスキー分解を使った表記](http://qiita.com/kilometer/items/1a95fa56aa0c90f56f04)

# 2. R code

これは、{rstan}経験者であれば特に問題ないでしょう。
最初のセクションで用意したデータを`list`にしておいて、`fit`関数に保存した.stanファイル名を指定するだけです。

```r
library("rstan")
rstan_options(auto_write=TRUE)
options(mc.cores=parallel::detectCores())

dat <- list(N = N, x = x, y = y, N2 = N2, x2 = x2)
fit <- stan(file = "gp.stan", data = dat, seed =123,
            chain = 3, iter = 2000, warmup = 500)
```
2-3行は、並列計算のおまじない。
`rstan::stan`のオプションはさっさと計算するための指定。
手元の環境で、上の設定だと1chainあたり200-700秒ぐらい(遅)。
色々警告が出ますが、収束しました。

可視化は、まぁ、こんな感じでしょうか。

```r
A <- extract(fit)

y2_med <- apply(A$y2, 2, median)
y2_max <- apply(A$y2, 2, quantile, probs = 0.05)
y2_min <- apply(A$y2, 2, quantile, probs = 0.95)

dat_g <- data.frame(x2, y2_med, y2_max, y2_min)
dat_g2 <- data.frame(x, y)

ggplot(dat_g, aes(x2, y2_med))+
  theme_classic()+
  geom_ribbon(aes(ymax = y2_max, ymin = y2_min), alpha = 0.2)+
  geom_line()+
  geom_point(data = dat_g2, aes(x, y))+
  xlab("x") + ylab("y")
```
<img width = 400, src = md/{rstan} Rstanでガウス過程の実装/527dff39-41da-df61-7b90-6442e5483cac.png>

# 環境

```r
sessionInfo()

> R version 3.4.0 (2017-04-21)
> Platform: x86_64-apple-darwin15.6.0 (64-bit)
> Running under: macOS Sierra 10.12.4
> 
> attached base packages:
> [1] stats     graphics  grDevices utils     datasets  methods   base     
> 
> other attached packages:
> [1] rstan_2.15.1         StanHeaders_2.15.0-1 ggplot2_2.2.1       
> 
> colorspace_1.3-2
> [17] gridExtra_2.2.1  tibble_1.3.1
```

