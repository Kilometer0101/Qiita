# 前振り
先に、{biOps}によるフィルタ操作を[書きました](qiita.com/kilometer/items/80dacc83380355cef26e)。

[前回](http://qiita.com/kilometer/items/bbd7d2b2a3e109be709a)、{EBImage}と{biOps}を使った画像生成を比較しました。
[0, 255]しか扱えない**imagedata**クラス(biOps)と、
数値なんでも扱える**image**クラス(EBImage)に注意ですね。

計算処理には{EBImage}が向いていますが、その代わり出来合いの関数が少ない。
なので道具を自分で作らないといけない事になる。

今回はその例としてCanny Edge detector を作ります。

# 罠
ただし、ですね。
{EBImage}は方向が変なんですよ。いや、変じゃないんですが…。
<img width=300 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/427b2599-bf3e-5be1-8116-ebec4fbf9d60.png">
こうなっているんです。

これでフィルタ処理とかすると、X軸・Y軸、プラスマイナスがカオスになって、
**大混乱**のうえ**悶え苦しむ**ので、絵を保存するときは-90度回転させます。
angleはdgreeで入れるのが何とも**むず痒い**。
`rotate(image, angle = -90)`

# サンプル画像
これは[前回](http://qiita.com/kilometer/items/bbd7d2b2a3e109be709a)のモノを使います。
3x3に拡大内挿(biliner法)したものを使います。
すでに画像になっているものを読み込むのは`readImage`。
これで`jpeg`, `tiff`, `png`, `bmp`など、代表的な形式は全て読めました。

```r
library("EBImage")
dat <- readImage("hoge.tiff")
```
<img width="300" src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/159c06ef-937a-c18f-d1fe-5de1730ad0be.png"> 


# Cannyの原理
[Canny Edge Detection -アルゴリズム-](http://d8yd.blog105.fc2.com/blog-entry-84.html)
こちらのサイトが良くまとまっておりました。

>Canny Edge Detectionのアルゴリズムは次の手順。
STEP1:ガウシアンフィルタで平滑化
STEP2:エッジ強度と勾配方向(4方向に量子化)を計算
STEP3:細線化処理（Non-maximum Suppression）
STEP4:ヒステリシス閾処理（Hysteresis Threshold）

これにのっとって進めましょう。


# 1. ガウシアンフィルタ
以前にも紹介しましたが、
[ガウシアンフィルタの特徴](http://imagingsolution.blog107.fc2.com/blog-entry-165.html)
ここのサイトが素晴らしくまとまっています。
要点は、ノイズは高周波成分なので、それをカットしようという発想。

画像に自分で設計したフィルタをかける際、
{biOps}では、`imgConvolve(imagedata, filter)`を使いました。
{EBImage}では、`filter2(image, filter)`とします。挙動は同じです。

以前と同様に設計からしてもいいのですが、{EBImage}では、ガウシアンフィルタ用の関数`gblur`が用意されています。

```r
dat.gau <- gblur(dat, sigma=2)
```
sigmaはフィルタに使う正規分布の標準偏差です。
<img width="300" alt="スクリーンショット 2016-01-29 12.26.09.png" src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/b576081a-53bf-8014-340b-aec2a91b12fd.png">

# 2-1. エッジ強度
エッジ強度の算出には、sobelフィルタを使います。
X, Y軸方向差分と似たような感じですね。

<img src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/b9db10dd-5f53-2466-4e03-a7b2399be460.png" width=200>

```r
sobel_x <- matrix(c(1,2,1, 0,0,0, -1,-2,-1), 3)
sobel_y <- matrix(c(-1,-2,-1, 0,0,0, 1,2,1), 3, byrow=T)
```
え〜と、正負の組み合わせが後の計算に影響するポイントです。
この組か、x, yどちらも正負逆転した組ならOKでしょう。

フィルタをかけます。

```r
dat.sobelx <- filter2(dat.gau, sobel_x)
dat.sobely <- filter2(dat.gau, sobel_y)
```
 <img width=300 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/cee4c3be-d052-172c-14ac-ddc4cc82e252.png">


で、ここが{EBImage}を使う利点ですが、これには負の値も含まれています。
`range(dat.sobelx)`は、`-0.86 0.86`とかになります。

そうすると、こーゆー事ができます。

```r
dat.edgeAbs <- sqrt(dat.sobelx^2 + dat.sobely^2)
```

<img width=200 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/c2f7f5da-61d9-ce31-a639-7770f81eb29f.png">

<img width=300 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/8759eee3-468d-b825-0ae1-2ee21d4a0f6d.png">

これを{biOps}でやろうとすると、差分画像が[0, 255]になっちゃうので、
2乗すると意味不明になります。しかもこんな気軽に四則演算は出来ません。

# 2-2. エッジ勾配方向
勾配方向は、tangentで求めます。

```r
dat.edgeTan <- dat.sobely / dat.sobelx
```
で一瞬。
これは、エッジの向きに対して垂直になるはずですね？
<img width=250 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/09c38b7d-83a9-a302-9191-c26e7f03b582.png">

これを水平・鉛直・45度・-45度の4段階に離散化します。
格好良く書けるかもしれませんが、まぁ、**ゴリ押し**で十分でしょう。

```r
A <- tan(pi/8)
B <- tan(pi*3/8)

dat.edgeTan2 <- as.Image(matrix(0, nrow(dat.edgeTan), ncol(dat.edgeTan)))
dat.edgeTan2[dat.edgeTan > -A  & dat.edgeTan <=  A ] <- 0    # 水平
dat.edgeTan2[dat.edgeTan >  A  & dat.edgeTan <   B ] <- 0.3  # 45度
dat.edgeTan2[dat.edgeTan > -B  & dat.edgeTan <= -A ] <- 0.6  # -45度
dat.edgeTan2[abs(dat.edgeTan) >= B ] <- 1   # 鉛直
```
<img width=300 src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/5f48174a-f7e8-52a1-7849-faec292e5508.png">

エッジの勾配(垂線方向)が、指定の4色で塗り分けられています。


# 3. 細線化処理（Non-maximum Suppression）
ガウシアンフィルタでぼやけているエッジを細くします。

<img src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/5c9c784e-e83c-afed-5c34-d70fbd76236a.png" width=600>

となっているわけですので、**エッジに対して垂直方向の隣接**ピクセルを拾ってきて、
**その中でmax**になっていればエッジ、そうでないものを捨てます。

どうやって隣接ピクセルを拾いますか？
これも、恐らく格好良く書けるんでしょうけれど…。
原始的な発想では、
<img width="100" alt="スクリーンショット 2016-01-29 14.36.44.png" src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/92fa6d6e-aa74-a37d-271c-7701f71bf34b.png">
この2つのフィルタを被せて、両方で正の値を取っていれば、隣接水平方向でmaxですね。

それでも、少しだけ格好良くやりたい。

```r
# 水平フィルタ
fil <- matrix(0, 3,3)
fil[2,2] <- 1
fil_1.1 <- fil; fil_1.1[2,3] <- -1
fil_1.2 <- fil; fil_1.2[2,1] <- -1

# 2.1で作ったエッジ強度に、水平フィルタをかける
dat1.1 <-  filter2(dat.edgeAbs, fil_1.1)
dat1.2 <-  filter2(dat.edgeAbs, fil_1.2)

# 水平隣接中で最大値のpixelを1、他を0にしたマスクを作る
mask1 <- as.Image(matrix(0, nrow(dat1.1), ncol(dat1.1)))
mask1[dat1.1 > 0 & dat1.2>0] <- 1

# 2.2で作った離散化エッジ勾配が 水平 == TRUE/FALSE
a <- dat.edgeTan2 == 0

# aの位置にだけmask1を適用する。
dat.edgeAbs2 <- dat.edgeAbs
dat.edgeAbs2[a] <- dat.edgeAbs2[a] * mask1[a]
```
<img src=md/{EBImage} Canny Edge Detectionを朴訥に書く。/4e85c032-8a26-e3ec-5588-9318f42106a7.png width=300>
という事で、水平方向の細線化ができました。

これを4方向（水平・鉛直・45度・-45度）にやります。
もちろん、格好良く出来るんでしょう。
しかし、私はそろそろ**自伝**を出そうと思っているぐらいの**コピペ**エンジニアなので、
今回も**嵐**のようなコピペで乗り切ります(従ってコードは割愛)。

<img src=md/{EBImage} Canny Edge Detectionを朴訥に書く。/2896271c-25cf-c976-8c1c-38c213c54cc7.png width=300>

# 4. ヒステリシス閾処理（Hysteresis Threshold）
個人的には、こんなに根性いれて2値化しなくていいんですけどねぇ。。

まぁ、しょうがない。この処理も組んでみましょう。

えーと、まず、2つの閾値でハイパス・ローパスフィルタを掛けます。
この閾値は**天から降ってきた値**です。

```r
high <- 0.35
low <- 0.25

dat2 <- dat.edgeAbs2
  dat2[dat2 >= high] <- 100
  dat2[dat2 <= low] <- 0
```
<img src=md/{EBImage} Canny Edge Detectionを朴訥に書く。/fc5b2b51-95bf-044b-0d7f-de1551d414d1.png width=300>

<img src=md/{EBImage} Canny Edge Detectionを朴訥に書く。/9ea0e2d4-ef92-9c4f-8c6e-6e0aeeb47cac.png width=200>



こうなっているので、**エッジに接続していればエッジ**にする。
では、**接続**ってどうすんだ。

いや要するに、
<img width="200" alt="スクリーンショット 2016-01-29 17.29.52.png" src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/7e8ae7df-7b1e-f57c-b80f-cded92af3f24.png">
こうなっていたら、ドウするんですかね。
この黒は**ワタッ**ているのか、**キレ**ているのか。囲碁みたいですね。

囲碁だと一見トビも繋がっているかもしれませんが、ここでは除きます。

まぁ、この場合は繋がっているとしましょう。
すると、周囲8マスにエッジがあればエッジとします。

先ほど、わざわざ

```r
  dat2[dat2 >= high] <- 100
```
としたのはこのためで、周囲8マスを合計するフィルタをかけて、100を超えていれば接続とみなしましょう。
これが1だと、エッジがなくても閾値を超えてしまう可能性がある。

フィルタは、真ん中だけ0で、他を1にすれば合計になります。
というか、真ん中1でもいいですよね、誤差みたいなもんです。

```r
fil_hist <- matrix(rep(1, 9), 3, 3)

repeat{
  mask_hist <- filter2(dat2, fil_hist)
    mask_hist[mask_hist < 100] <- 0
    mask_hist[mask_hist >= 100] <- 100

  dat3 <- dat2
    dat3[mask_hist !=0  & dat2 != 0] <- 100

  if(!all(dat2 == dat3)) dat2 <- dat3
  else break
}
```

mask_histは、「エッジもしくはエッジに8方のいずれかで隣接する点」。
dat3は、maskが0で無く、dat2の0で無い点を100にしたもの。
それが元のdat2に完全に一致するまで`repeat`をかける。

<img src="md/{EBImage} Canny Edge Detectionを朴訥に書く。/486c6290-3be8-dad3-fa57-5c1bf6785cc7.png" width=300>


２値化しておしまい。

```r
dat_canny <- dat2
  dat_canny[dat2 < 100] <- 0
  dat_canny[dat2 >= 100] <- 1
```
<img src=md/{EBImage} Canny Edge Detectionを朴訥に書く。/d5a6f302-cc04-f74a-2090-f9a5ccc1f91c.png width=300>

# まとめ
空から降ってきた値は、
・ガウス平滑化のパラメータ: **σ**
・閾値2種類： **high**, **low**
・接続の定義： 8方向で接続とみなす。

この辺りは、実際のデータと付き合わせて調整ではないですかね〜。

ただ、最後までやるなら`biOps::imgCanny`でおしまいで良いと思います。
私の場合、手で組んだのは、2値化する前の段階で止めたかったからです。

