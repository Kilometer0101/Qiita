# n変量正規分布

n変数正規分布の確率密度関数は、任意の$x\in{}\mathbb{R}^n$として、

```math
f(x) =f(x\ |\ μ, \Sigma) = \frac{1}{(\sqrt{2\pi})^{n}\ \sqrt{|\Sigma|}}\ exp\Big\{-\frac{1}{2}(x-μ)^T\Sigma^{-1}(x-μ)\Big\}  \tag{1}
```

(1)に従う集合$X$について、

```math
\vec{X} \sim N(\vec{μ}, \Sigma) \tag{2}
```
と書くが、この時、

```math
E(\vec{X}) = \vec{μ} ,\ var(\vec{X})=\Sigma \tag{3}
```

$\Sigma$は、$n\times{}n$行列で分散共分散行列。



# 条件付きn変量正規分布

```math
( \vec{X_1},\ \vec{X_2}) \sim{} N(\vec{μ}, \Sigma) \tag{4}
```

ならば、　

```math
\Sigma=\begin{bmatrix}\Sigma_{11} & \Sigma_{12} \\ \Sigma_{21} & \Sigma_{22}\end{bmatrix}
```
($\Sigma_{21}=\Sigma_{12}^T$ですね。)

として、

```math
\vec{X_2}\ |\ \vec{X_1} \sim{} N(\vec{μ_{2|1}},\ \Sigma_{22|1}) \tag{5}
```
ただし、

```math
\begin{eqnarray}
\vec{μ_{2|1}} &=& \vec{μ_2}+\frac{\Sigma_{21}}{\Sigma_{11}}(\vec{X_1}-\vec{μ_1}) \tag{6}\\
\Sigma_{22|1} &=& \Sigma_{22}-\Sigma_{21}\Sigma_{11}^{-1}\Sigma_{12} \tag{7}\\
\end{eqnarray}
```


と書ける。

# 小考
という訳で、この$\Sigma_{ij}$に対して、カーネル関数$k(x_i, x_j)$を設計するわけですね。

例えば、

```math
k(x_i,x_j)=\Sigma_{ij}=\eta^2exp\big(-\rho^2(x_i-x_j)^2\big)+σ^2δ_{ij}
```
($\delta_{ij}$はクロネッカのデルタ)

すると、既知の$X_1$を条件とした未知$X_2$の予測分布が得られる。
つまりこれって観測データに基づく非観測データの推定なので、(多変量正規分布を仮定した)回帰や分類に使える、という事。


# cf.
第12回 多変量正規分布,
http://www.eco.osakafu-u.ac.jp/osakafu-content/uploads/sites/9/2014/06/us-ln12.pdf

二変量正規分布の条件付き分布の解釈 
http://www.creativ.xyz/bivariate-normal-distribution-328

