[Stan公式](http://mc-stan.org/users/documentation/)のLanguage Manualの中でガウス過程について書かれた18章の手前勝手和訳です。できるだけ原文に忠実な構成を心がけていますが、断りなく意訳や意味の補足を加える場合があります。勉強ついでに翻訳しているだけなので、こなれた日本語になってないのはご勘弁を。誤訳などあればジャンジャンご指摘ください。18章の中身は全部進める予定ですが、ペースは手前勝手とさせてください。Qiitaはstanのシンタックスハイライトに対応していないのかー、というのが悩み。

2018/06/06    18.1まで公開。
2018/06/07    18.2まで公開。
2018/06/08    18.3.2まで公開。
2018/06/13    18.3.3まで公開。

# 18. Gaussian Processes

ガウス過程は連続確率過程(continuous stochastic processes )の一種であり、したがって、関数上での確率分布を与えるものとして解釈されうる。連続関数に対する確率分布は、荒っぽく言えば、個々の測定可能な入力のそれぞれに対応する無限個の確率変数の集合として捉えることができる。サポートできる関数の幅の広さが、ガウシアン・プライアを一般の多変量(非線形)回帰問題においてポピュラな手法たらしめている。

ガウス過程の定義における特徴は、有限個の入力点における関数値の混合分布を多変量正規分布として考えることにある。これにより、有限量の観察データからモデルを適合したり、新たな有限のデータ点に対して予測するといった操作が取り扱いやすくなる。

シンプルな多変量正規分布が「平均値ベクトル」と「共分散行列」という2つのパラメータで表記されるのに対し、ガウス過程は「平均関数」と「共分散関数」で定義されている点が異なる。この二つの関数(平均関数と共分散関数)は、与えられた入力ベクトルに基づいて2つのパラメータ(平均値ベクトルと共分散行列)を返す。この2つのパラメータが、入力点に対応する関数出力の平均値と共分散を与える。

Stanにおけるガウス過程のencodeでは、平均・共分散関数を実装してその結果をガウシアン形式でサンプリング分布に外挿する、もしくは、下記に示す特化した共分散関数を用いる。この形式でのモデリングは単刀直入であり、シミュレーション・モデル適合・事後分布推定に用いることができる。また、正規分布出力を持つガウス過程は、潜在ガウス過程(latent Gaussian process)上で周辺化し、尤度と事後分布を解析的に算出するためにガウス分布のコレスキー分解を適用する事で、効率的に実装できる。

この章では、まずガウス過程を定義した後、単変量・多変量・多変量ロジスティック回帰について、シミュレーション・ハイパーパラメータ推定・事後分布推定の基本的な実装法について解説する。ガウス過程は非常に一般的な手法であり、従って必然的にこの章ではいくつかの基本的なモデルに触れるに止め、詳細は[Rasmussen and Williams, 2006]を参照のこと。

## 18.1. Gaussian Process Regression


多変量($D$次元)ガウス過程回帰のための$N$個のデータ$x$と組みとなる出力$y$を考える。

```math
x = {x_1, ..., x_N} ∈ R^D\\
y = {y_1, ..., y_N} ∈ R
```

ガウス過程の定義では、入力$x$によって条件づけられる有限数の出力$y$の確率を下記のガウシアンで定める。

```math
y ∼ MultiNormal(m(x), K(x|θ)),
```

$m(x)$は要素$N$のベクトルであり、$K(x|θ)$は$N × N$の共分散行列である。
平均関数$m: R^{N×D}→R^N$は任意だが、共分散関数$K: R^{N×D}→R^{N×N}$は、任意の入力$x$に対して正定値行列を生成しなければならない[^1]。

この章で扱うことになるポピュラな共分散関数は、下記の累乗二次関数(exponentiated quadratic function)である。

```math
K(x|α,ρ,σ)_{i,j} =α^2exp \Big(−\frac{1}{2ρ^2} \sum_{d=1}^{D}(x_{i,d} −x_{j,d})^2\Big)+δ_{i,j} σ^2
```

$α, ρ, σ$は共分散関数を定義するハイパーパラメータであり、$δ_{i,j}$はクロネッカのデルタ(Kronecker delta function)である。すなわち$i=j$の時$1$、他の場合$0$をとる正方行列の事である。

(尚、ここでいう$i=j$はデータのインデックスが等しいを表しており、データの値が等しい$x_i=x_j$という意味ではない。)

また、このカーネルは2つの独立したガウス過程$f_1$と$f_2$の畳み込みの結果として考えてもよい。それぞれ下記のカーネル$K_1$と$K_2$を持つ。

```math
K_1(x|α,ρ)_{i,j} =α^2exp\Big(−\frac{1}{2ρ^2} \sum_{d=1}^D(x_{i,d} −x_{j,d})^2\Big)
```

and 

```math
K_2(x|σ)_{i,j} = δ_{i,j}σ_2,
```

この$K_2$は、$σ^2$を対角成分に加えることを意味するが、それは2つの独立した入力$x_i$と$x_j$が等しい場合に得られる行列の正定値性を担保するために重要である。統計的な用語では、$σ$はノイズ項のスケールに該当する。

ハイパーパラメータ$ρ$は、長さのスケールであり、このドメインで用いられるガウス過程のプライアによって表される関数の周波数成分に対応する。すなわち、$ρ$がゼロに近ければガウス過程は高周波関数になり、大きくなるにつれて低周波関数になる。

ハイパーパラメータ$α$は周辺化された標準偏差(marginal standard deviation)であり、ガウス過程によって表される関数の振幅を表す。特定の$α$に固定しておいて、同じ入力$x$について$f_1$プライアに基づくガウス過程で多数の描画を行って標準偏差を求めると、それが固定した$α$の値に等しくなる。

この累乗二次関数において入力$x_i$と$x_j$が含まれる唯一の項が、($D$次元)ベクトルの差$x_i - x_j$で表されている。これは、定常な共分散を持った確率過程を作り出す。例えば入力ベクトル$x$が、ベクトル$ε$によって$x + ε$になったとしても、出力における全ての組み合わせの共分散は変化しない。何故なら $K(x|θ) = K(x + ε|θ)$だからだ。

$Σ$に含まれているのは、単なる$x_i$と$x_j$のユークリッド距離である(例えば, 差$x_i-x_j$のL2ノルムなど)。この結果、確率過程における関数が滑らかになる。関数の変化量は、自由なハイパーパラメータ$α, ρ, σ$によって制御されている。

この距離の概念をユークリッド距離からマンハッタン距離(例えばL1ノルム)に変更する事は、関数を連続であるが滑らかでないものに変更することを意味する。

[^1]: ガウス過程は正の半定定値行列を生成する共分散関数に拡張することが可能だが、Stanはサポートしていない。これは結果の分布が非制約条件を持っていないからだ。

## 18.2. Simulating from a Gaussian Process

関数$f$の描画をガウス過程からシミュレートする最も単純なモデルからStanを使い始めてみる。実際には、モデルによる計算で、入力$x_n$に対する有限個の$y_n = f(x_n)$を計算する。

Stanモデルでは、平均関数と共分散関数を `transformed data`のブロックで定義し、$y$は最も単純な出力として多変量正規分布を用いる。具体的なモデルとして、前節で登場した累乗二次関数のハイパーパラメータは、それぞれ$α^2 = 1, ρ^2 = 1, σ^2 = 0.1$に固定する。更に、平均関数$m$は、常にゼロベクトルを返す関数として定義する$m(x)=0$。

それでは、ガウス過程のシミュレータを下記で実装してみる[^2]。

```stan
data {
  int<lower=1> N;
  real x[N];
}
transformed data {
  matrix[N, N] K;
  vector[N] mu = rep_vector(0, N);
  for (i in 1:(N - 1)) {
    K[i, i] = 1 + 0.1;
    for (j in (i + 1):N) {
      K[i, j] = exp(-0.5 * square(x[i] - x[j]));
      K[j, i] = K[i, j];
    }
  }
  K[N, N] = 1 + 0.1;
}
parameters {
  vector[N] y;
}
model {
  y ~ multi_normal(mu, K);
}
```

上記のモデルは、累乗二次カーネル関数 `cov_exp_quad` を用いることで更にコンパクトに実装することができる。

```stan
data {
  int<lower=1> N;
  real x[N];
}
transformed data {
  matrix[N, N] K = cov_exp_quad(x, 1.0, 1.0);
  vector[N] mu = rep_vector(0, N);
  for (n in 1:N)
    K[n, n] = K[n, n] + 0.1;
}
parameters {
  vector[N] y;
} model {
  y ~ multi_normal(mu, K);
}
```

入力するデータ長さ$N$のベクトル$x$だけである。このモデルは、等間隔で配置された$x$に対するガウス過程からサンプルされた関数を描画するために用いることができる。

### 18.2.1 Multivariate Inputs

入力データフォームを変えるだけで多変量データに基づく推定が可能だ[^2]。したがって、上記の単変量モデルのうち下記の部分を変更するだけ。

```stan
    data {
      int<lower=1> N;
      int<lower=1> D;
      vector[D] x[N];
    }
    transformed data {
    ...
    ...
```

ここではデータは数値アレイではなくベク通るアレイとしての定義に変更されている。また、データ次元$D$も定義されている。

簡単に振り返ると、単変量モデルは平易に使えるが、そのどんな例でも簡単に多変量モデルに拡張することができ、そこでのサンプリングモデルも同様にシンプルである。多変量への拡張によって生じる唯一の計算コストは距離計算の部分にある。


### 18.2.2 Cholesky Factored and Transformed Implementation

Stanでは、このモデルをより効果的に実装する方法として、等方正規単位(isotropic unit normal variate)を再配置・再スケール・回転する手法があります。$η$(`eta`)が等方正規単位であるとは、下記を示します。この$0$は長さ$N$の零ベクトルであり、$1$は$N × N$の単位行列である。

```math
η ∼ Normal(0, 1),
```

ここで、$L$を$K(x|θ)$のコレスキー分解(Cholesky decomposition)だとします。$L$は例えば、$LL^T= K(x|θ)$を満たす下三角行列と置ける($L^T$は逆行列)。すると、$μ + Lη$の分布は次の様に表記できる。

```math
μ + Lη ∼ MultiNormal(μ(x), K(x|θ)).
```

この変換は、ガウス過程のシミュレーションに直接的に適用することができる[^2]。この場合、`data`ブロックにおける$N$と$x$の宣言は同一であり、`transformed data`における$mu$と$K$も上と同様である。`transformed data`においてコレスキー分解を導入する。また`parameters`は、`y`から、より定時なパラメータ$η$とする。これは等方正規単位からサンプルされた量として定義される。

```stan
...
transformed data {
  matrix[N, N] L;
  ...
  L = cholesky_decompose(K);
}
parameters {
  vector[N] eta;
} model {
  eta ~ normal(0, 1);
}
generated quantities {
  vector[N] y;
  y = mu + L * eta;
}
```

このコレスキー分解は、データが読み込まれて共分散行列$K$が計算されれば、1回の計算で求まる。$η$のための等方正規分布は、効率化のためにベクトル化された単変量分布である。この記載によってそれぞれの`η[n]`が独立した単位正規分布からサンプルされることを指定している。推定値$y$は、多変量正規分布をコレスキー分解により変換する式をそのままコードする事によって`generated quantities`ブロックにおいて定義される事になる。

[^2]: The code is available in the Stan example model repository; see http://mc-stan.org/ documentation.



## 18.3. Fitting a Gaussian Process 

### 18.3.1 GP with a normal outcome

正規分布に従う出力を持つガウス過程の最も一般化された実装は、出力$y ∈ R^N$, 入力$x ∈ R^N$として、有限の$N$に対して下式で与えらえる。

```math

\begin{eqnarray}

ρ &∼& InvGamma(5, 5) \\
α &∼& Normal(0, 1)\\
σ &∼& Normal(0, 1)\\
f &∼& MultiNormal(0,K(x|α,ρ))\\
y_i &∼& Normal(f_i,σ)\quad ^{∀}i ∈ {1,...,N}
\end{eqnarray}
```

出力$y$が正規分布に従う場合、ガウス過程の出力$f$と統合し、もう少し簡単な形でモデルを書くことができる。

```math
\begin{eqnarray}
ρ &∼& InvGamma(5, 5) \\
α &∼& Normal(0, 1)\\
σ &∼& Normal(0, 1)\\
y &∼& MultiNormal\big(0,K(x|α,ρ)+I_{N}σ^2\big)
\end{eqnarray}
```

正規分布に従う出力を取り扱う時、出力にガウス過程を統合することで、より低次元のパメータ空間に落とし込めるので、計算コストを下げることが可能だ（どちらのモデルもstanでfitできる）。前者のモデルを潜在変数ガウス過程(lattent variable GP)と呼び、後者を周辺尤度ガウス過程(marginal likelihood GP)と呼ぶ。
ガウス過程の共分散関数を制御しているハイパーパラメータ(3種)は、我々が上記モデル式で示したそれぞれの事前分布から抽出され、観測データにより事後分布が与えられる。これらのパラメータの事前分布は、それぞれの事前の知識（$α$：出力値のスケール, $σ$：出力ノイズのスケール, $ρ$：入力間距離の尺度スケール）に基づいて定義される必要がある。18.3.4節にこれらのハイパーパラメータの事前分布を適切に特定するための詳しい情報を記載した。

#### Marginal likelihood GP
Stanは周辺尤度GPを下記のように実装している。このプログラムは前述したGPの推定に似ているが、ハイパーパラメータにも推定を行うために、共分散行列$K$の計算を`transformed data`ブロックではなく`model`ブロックで行う必要がある。[^2]


```stan
data {
  int<lower=1> N;
  real x[N];
  vector[N] y;
}
transformed data {
  vector[N] mu = rep_vector(0, N);
}
parameters {
  real<lower=0> rho;
  real<lower=0> alpha;
  real<lower=0> sigma;
} model {
  matrix[N, N] L_K;
  matrix[N, N] K = cov_exp_quad(x, alpha, rho);
  real sq_sigma = square(sigma);
  // diagonal elements
  for (n in 1:N)
    K[n, n] = K[n, n] + sq_sigma;
  L_K = cholesky_decompose(K);
  rho ~ inv_gamma(5, 5);
  alpha ~ normal(0, 1);
  sigma ~ normal(0, 1);
  y ~ multi_normal_cholesky(mu, L_K);
}
```

ここでは、`data`ブロックは入力`x[n]`に対する観測値ベクトル`y[n]`を宣言している。そして`transformed data`ブロックでは平均値関数`mu=0`を宣言しているだけだ。3つのハイパーパラメータは`parameter`ブロックにおいて負にならない実数として宣言される。共分散行列$K$の計算は未知数が含まれるため`transformed data`ブロックで事前に計算できないため、`model`ブロックに書かれる。`model`ブロックの残りの部分では、ハイパーパラメータの事前分布が与えられ、また、多変量正規分布のコレスキー化された尤度(multivaliate Cholesky-parameterized normal likelihood)の実装になっている。ここで、
$y$が基地である事から、共分散行列$K$は未知であるがハイパーパラメータに依存しないため、結果としてハイパーパラメータを推定する事が可能になる。

ここでは、（$L$について）行列サイズに関してより汎用性が高い`cholesky_decompose`を用いるために、（$y$について）通常の多変量正規`multi_normal`ではなくコレスキー化された多変量正規分布`multi_normal_cholesky`を使った。小さな行列を対象に計算する場合、両者の差はあってないようなものだが、大きな行列($N≥100$)を計算する場合、コレスキー分解を用いる版の方が高速である。

ハミルトニアン・モテカルロ法（Hamiltonian Monte Carlo sampling）はこのモデルにおいてハイパーパラメータの高速かつ効率的な推定を実現する(Neal, 1997)。これらの事後分布がよく収束している場合、数百のデータポイントに対するハイパーパラメータのfitを秒のオーダーで実行できる。


#### Latent variable GP
Stanでは潜在変数ガウス過程も明快に実装することができる。これは出力が非正規分布出会った時に有効だ。正の成分$δ$ `delta`を共分散行列の対角成分に追加することで、共分散行列が正である事を保証する必要がある。

```stan
data {
  int<lower=1> N;
  real x[N];
  vector[N] y;
}
transformed data {
  real delta = 1e-9;
}
parameters {
  real<lower=0> rho;
  real<lower=0> alpha;
  real<lower=0> sigma;
  vector[N] eta;
} model {
  vector[N] f;
  {
    matrix[N, N] L_K;
    matrix[N, N] K = cov_exp_quad(x, alpha, rho);
    // diagonal elements
    for (n in 1:N)
      K[n, n] = K[n, n] + delta;
    L_K = cholesky_decompose(K);
    f = L_K * eta; }
  rho ~ inv_gamma(5, 5);
  alpha ~ normal(0, 1);
  sigma ~ normal(0, 1);
  eta ~ normal(0, 1);
  y ~ normal(f, sigma);
}
```

周辺尤度GPと(潜在変数GPと)の2つの違いに着目して解説する。
1つ目のポイントは、`parameter`ブロックにおいて、新たな長さ$N$のパラメータベクトル$η$が追加された。これは`model`ブロックにおいて多変量正規ベクトル$f$(これが潜在ガウス過程に相当する)を生成するために用いられる。$η$に対しては、前述のコレスキー化したGPの実装と同様に$Normal(0, 1)$を事前分布として与える。
2つ目は、ここでは尤度が1次元になっているという点だ。$σ^2$からなる単位共分散行列を用いることで、N個の尤度を1つのN次元多変量正規分布としてコードできるにも関わらず、この実装を行うのは、上記のようにベクトル化した方が計算コスト上有利だからだ。


### 18.3.2 Discrete outcomes with Gaussian Processes

ガウス過程は通常の線形回帰モデルと同様に、リンク関数を用いることで拡張できる。これによって離散量データのモデリングに用いることができる。

#### Poisson GP

カウントデータをモデル化する場合、$σ$パラメータを用いず、尤度関数に`normal`ではなく対数リンク関数`poisson_log`を用いる。また、$y$の期待値を周辺化するために全体平均パラメータ$a$を導入する。これはカウントデータでは中央値を正規分布の様に設定する事ができないためだ。

```stan
data { ...
  int<lower=0> y[N];
  ...
}
...
parameters {
  real<lower=0> rho;
  real<lower=0> alpha;
  real a;
  vector[N] eta;
} model { ...
  rho ~ inv_gamma(5, 5);
  alpha ~ normal(0, 1);
  a ~ normal(0, 1);
  eta ~ normal(0, 1);
  y ~ poisson_log(a + f);
}
```

#### Logistic Gaussian Process Regression

2値分類問題においては、観察データ$z_n ∈ {0, 1}$はバイナリで与えられる。この出力は非観察出力$y_n$のガウス過程としてロジスティックリンク関数を用いてモデル化できる。

```math
z_n ∼ Bernoulli(logit^{−1}(y_n))
```

もしくは、

```math
Pr[z_n = 1] = logit^{−1}(y_n)
```

と表せる。

これに対処するため、潜在変数GPのstanプログラムを拡張することができる。下記は、教師データのクラス不均衡性を補正するbias項である。

```stan
data { ...
  int<lower=0, upper=1> z[N];
  ...
}
...
model {
  ...
  y ~ bernoulli_logit(a + f);
}
```


### 18.3.3 Automatic Relevance Determination

入力が多変量$x ∈ R^D$である場合、累乗二次共分散関数に対して、各次元$d$に対応するスケール項$\rho_d$でフィッティングするという拡張を行う。

```math
k(x|α,ρ⃗,σ)_{i,j} =α^2 exp \Big( -\frac{1}{2} \sum_{d=1}^{D} \frac{1}{ρ_d^2}(x_{i,d}-x_{j,d})^2\Big)+δ_{i,j}σ^2
```

推定量$ρ$は、関連度自動評価系数(automatic relevance determination, ARD)と呼ばれているが(Neal, 1996a)、この名称は誤解を招く。何故なら、個々の$\rho_d$の事後分布の振幅は、入力データの次元$d$におけるスケールに依っているからだ。この$\rho_d$のスケールは関連性というよりも次元$d$における非線形性を測定しているものだ(Piironen and Vehtari, 2016)。
事前分布においては$\rho_{d}$がゼロに近ければ近いほど次元$d$における条件付き平均がより非線形である事を表し、事後分布においては実際の$x$と$y$の関連性がその役割を果たす。すなわち、ある共変量$x_1$が線形の効果を持ち、別の$x_2$が非線形の効果を持つとして、$x_1$の方が予測関連性が例え高かったとしても、$\rho_1 > \rho_2$となりうるのは問題だ(Rasmussen and Williams, 2006, page 80)。集合$\rho_d$もしくはその逆数($1/\rho_d$)を階層的にモデル化する事も可能だ。
StanにおいてARDは率直に実装できるが、現時点ではユーザーが共分散行列を直接コードする必要がある。ここでは共分散行列のCholesky分解を生成する`L_cov_exp_quad_ARD`関数を紹介する。

```stan
functions {
  matrix L_cov_exp_quad_ARD(vector[] x,
                            real alpha,
                            vector rho,
                            real delta) {
    int N = size(x);
    matrix[N, N] K;
    real sq_alpha = square(alpha);
    for (i in 1:(N-1)) {
      K[i, i] = sq_alpha + delta;
      for (j in (i + 1):N) {
        K[i, j] = sq_alpha
        * exp(-0.5 * dot_self((x[i] - x[j]) ./ rho));
        K[j, i] = K[i, j];
      }
    }
    K[N, N] = sq_alpha + delta;
    return cholesky_decompose(K);
  }
} 
data {
  int<lower=1> N;
  int<lower=1> D;
  vector[D] x[N];
  vector[N] y;
}
transformed data {
  real delta = 1e-9;
}
parameters {
  vector<lower=0>[D] rho;
  real<lower=0> alpha;
  real<lower=0> sigma;
  vector[N] eta;
} model {
  vector[N] f;
  {
    matrix[N, N] L_K = L_cov_exp_quad_ARD(x, alpha, rho, delta);
    f = L_K * eta; }
  rho ~ inv_gamma(5, 5);
  alpha ~ normal(0, 1);
  sigma ~ normal(0, 1);
  eta ~ normal(0, 1);
  y ~ normal(f, sigma);
}
```

### 18.3.4 Priors for Gaussian Process Parameters
#### Priors for length-scale
#### Priors for marginal standard deviation
### 18.3.5 Predictive Inference with a Gaussian Process
#### Predictive Inference in non-Gaussian GPs
#### Analytical Form of Joint Predictive Inference
### 18.3.6 Multiple-output Gaussian processes

