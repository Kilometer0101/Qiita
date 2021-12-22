{biOps}インストールについては[前々回](http://qiita.com/kilometer/items/772e355b19ac810f71a6)を参照。

サンプル画像については[前回](http://qiita.com/kilometer/items/4f48bd50dd47cf6724c7)を参照。
ただし、今回はノイズが多めに載ってます。
[0, 90]の一様乱数。他は前回と同じ。

# ガウシアンフィルタ

結果から。
![gaussfilter_1.png](md/{biOps} 画像のフィルタ操作/01584d43-a8f7-6f44-95f5-9c6926d805b1.png)
順調に背景ノイズが緩和されています。
が、細い線はブロードになります。


{biOps}では、

```r
imgConvolve(imgdata, mask)
```

として、maskの中にフィルタをmatrixで書く。
Gaussian filterの事は、[この素晴らしいページ](http://imagingsolution.blog107.fc2.com/blog-entry-165.html)に全て書いてあります。
要するに、ノイズはシグナルに比べて高周波でバラバラと入るわけだから、その成分をカットすれば良いという発想です。

Rで書くには、

```r
# これはσ ≒ 1.0の時
a <- c(1,4,6,4,1)
b <- NULL
for(i in 1:length(a)) b <- cbind(b, a*a[i])
fill_Gauss <- b/sum(b)

imgConvolve(imgdata, fill_Gauss)
```

という感じでゴリ押しで書けばよいだけ。

{biOps}では、

```r
plot(imgdata)
```
で、勝手に絵が出ます。


# 微分フィルタ

別にどうということもなく、ですが。
Y軸方向微分ですと、

```r
fil_y1 <- matrix(rep(c(-1,0, 1), 3), 3)
fil_y2 <- matrix(rep(c( 1,0,-1), 3), 3)
```
の2種類があるわけです。

```r
byrow=T
```
とすればX軸方向になりますね。


```r
y1  <- imgConvolve(dat, fil_y1)
y2  <- imgConvolve(dat, fil_y2)
y12 <- imgAdd(y1, y2)
```
という感じにすると、
![y_filter1.png](md/{biOps} 画像のフィルタ操作/5ccd61b7-8066-796c-7803-edcdb7de0df8.png)

で、これをX軸にもやって、mergeするわけです。
![xy_filter.png](md/{biOps} 画像のフィルタ操作/d2d5d881-88cf-133e-e6cc-7e97c3987b1e.png)

ふむ、で、最初にGaussian filter(σ≒1.3)をかけておくと、
![xyGau_filter.png](md/{biOps} 画像のフィルタ操作/3af541cf-2c5a-6a56-84d9-66aa6cf1fbc2.png)

最後に、これ。
オリジナル、splineで3x3に拡張補間、Gaussian filter(σ≒1.3)、差分。
![xy_sp_Gau_filter.png](md/{biOps} 画像のフィルタ操作/665df5d4-023f-1c4c-52a7-b2fa6cd6ba58.png)


# Canny Edge ditector
まぁ、出来合いのを使えばサクッと。

```r
dat <- readTiff("hoge")
dat3 <- imgScale(dat, 3, 3, interpolation="spline")
dat_Canny <- imgCanny(dat3, sigma=1)
```
![canny1.png](md/{biOps} 画像のフィルタ操作/2bd146f3-a176-6ed4-4ea5-3701caace220.png)

Cannyフィルタは、Gaussianフィルタを掛けて差分をとって、二値化をする。
この二値化がやたら凝っていて、ちゃんとエッジになるらしい。
詳しくはグーグる。

このsigmaは、Gaussのsigmaなので変えると輪郭も変わる。

```r
par(mar=c(0,0,0,0), mfrow=c(4,4))
for(sigma in seq(0.5, 2, by=0.1)){
  dat_Canny <- imgCanny(dat3, sigma=sigma)
  plot(dat_Canny)
}
```

![canny2.png](md/{biOps} 画像のフィルタ操作/7f769693-fc4c-3f76-2b68-c005675a2f28.png)

という事で、最後までゴリ押しでした。

{EBImage}については、まぁ、そのうちに。

