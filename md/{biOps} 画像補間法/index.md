{biOps}のインストールについては、[前記事参照](http://qiita.com/kilometer/items/772e355b19ac810f71a6)の事。

# サンプル画像の作成

背景は[0, 40]の一様乱数で、カクカク曲線と直線が交差しているようにする。
![sample2.png](md/{biOps} 画像補間法/a2e459c2-8a87-cd7b-f1c7-7ad7d57a893c.png)
おお、それっぽい。

```r
# 乱数のシード
set.seed(11)
# 円のxy座標を整数値で吐き出す。
x <- round(20*cos(seq(0, 2*pi, by=0.05)) + 31)
y <- round(20*sin(seq(0, 2*pi, by=0.05)) + 31)
xy <- cbind(x,y)
# 乱数に縦線と横線。
a <- matrix(runif(61*61, 0, 40), 61)
a[31,] <- 150
a[,29:31] <- 100
# ゴリ押しで円の部分を白抜き
for(i in 1:nrow(xy)){
  a[xy[i,1], xy[i,2]] <- 255
}

# 画像を保存-その1
dat <- imagedata(a)
writeTiff("sample.tiff", dat)

# 画像を保存-その2
tiff("sample.tiff", width=61, height=61)
par(mar=c(0,0,0,0))
image(1:nrow(a), 1:ncol(a), a, col=grey(0:255/255))
dev.off()

```

# Let's 内挿

![スライド1.png](md/{biOps} 画像補間法/27225172-0a9b-2da8-23e4-a7148806f935.png)
全体像
![スライド2.png](md/{biOps} 画像補間法/3c835583-aeec-b717-adc6-e0f1e9b50d2a.png)
拡大像

```r
library("biOps")

dat0 <- readTiff("sample.tiff")
dat1 <- imgScale(dat0, 3, 3, interpolation = "bilinear")
dat2 <- imgScale(dat0, 3, 3, interpolation = "cubic")
dat3 <- imgScale(dat0, 3, 3, interpolation = "spline")

writeTiff("hoge.tiff", dat)
```


# で、何なのか。

これは、要するに、
![スライド1.png](md/{biOps} 画像補間法/1dafb4a0-1a01-066f-2070-6ea4e6aaa08d.png)
という事をして、隙間を自然に埋めよう、という事。なので、
![スライド2.png](md/{biOps} 画像補間法/2ee11c10-40c5-012e-20f0-076ed7bfcd28.png)
こんな事を考えて、赤い線の長さがわかれば良い。
![スライド3.png](md/{biOps} 画像補間法/8bd5807b-279e-ef39-cb4c-2d6a0b4953a0.png)
こうですね。

この補間関数に何を使うか、という話。

**biliner**では、近傍4点の重回帰。超シンプル。

**cubic**では、近傍16点で3次関数でフィッティング。
これは、4点あれば3次関数が1義に決まるから。
一般にはbicubic法と呼ばれる。

**spline**では、B-splineという関数でフィッティングする。
BはBasicのBらしい。詳しくはグーグル。

結果は見ての通り。
bicubicは結構クセがありますねぇ。
背景ノイズに強いのはB-splineでしょうか。






