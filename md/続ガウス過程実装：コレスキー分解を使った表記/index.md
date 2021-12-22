承前
・[ガウス過程の考え方](http://qiita.com/kilometer/items/6ab1d7f2cffc0c6e4a95)
・[{rstan}Rstanでガウス過程の実装](http://qiita.com/kilometer/items/8b81560c0efef5e0cee2)

参考
・[ガウス過程シリーズ2 高速化&フルベイズ](http://statmodeling.hatenablog.com/entry/gaussian-process-2)
・[コレスキー分解:wikipedia](https://ja.wikipedia.org/wiki/%E3%82%B3%E3%83%AC%E3%82%B9%E3%82%AD%E3%83%BC%E5%88%86%E8%A7%A3)
・[多変量正規分布:統数研](http://random.ism.ac.jp/info01/distribution_random_number_generation/node2.html)

# コレスキー分解の活用

[{rstan}Rstanでガウス過程の実装](http://qiita.com/kilometer/items/8b81560c0efef5e0cee2)の、1-6節にあたる部分です。

入力`x`に対する出力`y`を条件として、入力`x2`に対する未知の出力`y2`を、条件付き多変量正規分布として求めたい。yが正規化されているとして、

```math
y \sim N_{n_1}(0, cov) \\
```

```math
\begin{bmatrix}y \\ y_2\end{bmatrix} \sim 
N_{n_1+n_2} \Bigg(0,\begin{bmatrix}cov & K \\ K^t &\Sigma\end{bmatrix} \Bigg)
```

を条件とした多変量正規分布を考え、

```math
\begin{eqnarray}
y_2 &\sim& N_{n_2}(\mu_2, cov_2)\\
\mu_2&=&K^tcov^{-1}y\\
cov_2&=&\Sigma-K^tcov^{-1}
\end{eqnarray}
```

これを元に、stanで`multi_normal_rng`で乱数生成する事で、`y2`の範囲を推定する事も可能なのですが、多変量正規乱数の生成がヘビィなので、`normal_rng`で済ませたい。

$cov_2$は、実対称行列なので、下三角行列$L$を用いて、

```math
cov_2 = LL^t\tag{2-1}
```

と分解できる。(これがコレスキー分解)

分解できると何が良いかというと、$y_2 \sim N_{n_2}(\mu_2, cov_2)$は、互いに独立な**標準正規乱数**の集合$z \sim N(0,1)$を用いて、

```math
y_2\sim \mu_2+Lz\tag{2-2}
```

と書ける。
これを使ってstan codeを全部まとめると、


```stan
data{
  int<lower = 1> N;
  vector[N] x;
  vector[N] y;
  
  int<lower = 1> N2;
  vector[N2] x2;
}

transformed data{
  vector[N] mu;
  for(i in 1:N)
    mu[i] = 0;
}

parameters {
  real<lower = 0> eta_sq;
  real<lower = 0> rho_sq;
  real<lower = 0> sigma_sq;
}

transformed parameters{
  matrix[N, N] Cov;
  
  for(i in 1:(N-1)){
    for(j in (i+1):N){
      Cov[i, j] = eta_sq * exp(-rho_sq * pow(x[i] - x[j], 2));
      Cov[j, i] = Cov[i,j];
    }
  }
  
  for(k in 1:N)
    Cov[k, k] = sigma_sq;
}

model{
  y ~ multi_normal(mu, Cov);
  
  eta_sq ~ cauchy(0, 5);
  rho_sq ~ cauchy(0, 5);
  sigma_sq ~ cauchy(0, 5);
}


generated quantities{
  vector[N2] y2;
  vector[N2] mu2;
  matrix[N2, N2] Cov2;
  matrix[N, N2] K;
  matrix[N2, N2] Sigma;
  matrix[N2, N] K_t_Cov;

  vector[N2] z;
  matrix[N2, N2] L;
  
  for (i in 1:N)
    for (j in 1:N2)
      K[i, j] = eta_sq * exp(-rho_sq * pow(x[i] - x2[j],2));

  for(i in 1:(N2-1)){
    for(j in (i+1):N2){
      Sigma[i, j] = eta_sq * exp(-rho_sq * pow(x2[i] - x2[j], 2));
      Sigma[j, i] = Sigma[i, j];
    }
  }

  for(k in 1:N2)
    Sigma[k, k] = sigma_sq;

  K_t_Cov = K' / Cov;  
  mu2 = K_t_Cov * y;
  Cov2 = Sigma - K_t_Cov * K;

// 式(2-1)  
  L = cholesky_decompose(Cov2);

  for(i in 1:N2)
    z[i] = normal_rng(0, 1);

// 式(2-2)
  y2 = mu2 + L * z;
}
```

っと、速さはあまり変わらないですね。
これは、コレスキー分解&標準正規乱数を生成を繰り返し行う vs. 多変量正規乱数を生成するというお話だからですかね。

しかし、カーネル関数のハイパーパラメータ達が既知であれば、これは爆速になるそうな…。
と言ってもstanではGPは遅いと巷で噂だそうです。

<img width = 400, src = md/続ガウス過程実装：コレスキー分解を使った表記/371e7e45-0bbd-4907-66f1-0fbc83b157b2.png>


