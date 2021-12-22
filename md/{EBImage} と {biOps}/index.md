# イントロ

先に紹介した**{biOps}**はエッジ検出周りの関数が非常に充実しています。
[rimagebook - FunctionList.wiki](https://code.google.com/archive/p/rimagebook/wikis/FunctionList.wiki)
ところが、**{biOps}**で画像を取り扱う**imagedata**クラスでは、勝手に[0, 255]に変換されてしまいます。なので、微分画像などを作っても負の値を取らなかったりして、計算過程を組む際の自由度が、低いんです。

**{EBImage}**では、**image**クラスで画像を取り扱います。これは数値を数値のまま取り扱えるので、例えば何らかの測定値を画像表現して概要を掴みながら、そのまま数値計算に落とせます。その代わり、自分で道具を用意しないといけない事が多い。

なので、出来合いの道具でよければ{biOps}、**変な事**をしたければ{EBImage}ですね。

あ、この2つのクラスを変換できる関数が、{RImageBook}に用意されているようですが、手元の環境ではインストールできず(バージョンの問題かも)。

というワケで、まずは{EBImage}での画像の生成関係。

# インストール
[EBImageを使った画像処理](http://d.hatena.ne.jp/Rion778/20091210/1260414280)
[EBImageのインストール方法](https://code.google.com/archive/p/rimagebook/wikis/EBImageInstallation.wiki)
の辺りを参照。

手元の環境では、R上で、

```r
source("http://bioconductor.org/biocLite.R")
biocLite("EBImage")
```
だけでインストールされました。
**何かの拍子**にImageMagicとgtkを入れていれば問題ないと思います。

# imageクラスの生成

数値matrixであれば何でもOKです。
ただし、`display(image)`で表示されるのは、[0,1]が最大256段階のグラデーション、1以上は255になります。えーとつまり、

```r
library("EBImage")
dat <- as.Image(matrix(seq(0, 2, length=100), 10, 10))
display(dat)
```
<img width="300" alt="スクリーンショット 2016-01-28 17.22.02.png" src="md/{EBImage} と {biOps}/a1a57381-6b94-9a57-ad1d-2dde1ff00c0c.png">
こんな感じですね。後半は1より大なので、真っ白。
同様に、負の値は真っ黒になります。
これは画像表示がこうなるって話で、数値は元のまま保持されdatに入っています。
あ、`display`は、ブラウザ上で画像が表示されます。

~~{biOps}にはimagedataクラスをmatrixから作る方法は(恐らく)無いので、
やるなら、matrixを`image`で書くのを`tiff`かなんかで一旦保存しておいて、`readTiff`で呼び出す。面倒ですね。~~

`(2016.01.29)`
ありました。記事を修正します。
`imagedata`ですが、これはmatrixの数値を[0, 255]で値を指定する必要があります。
なので、seq[0, 2]のままだと真っ黒になります。

```r
library("biOps")
dat <- matrix(seq(0, 2, length=100), 10, 10)

dat.img <- imagedata(dat*255/max(dat))
plot(dat.img)
```
<img width="300" alt="スクリーンショット 2016-01-29 10.19.29.png" src="md/{EBImage} と {biOps}/d639664e-ccb9-3d29-83c9-eca448e8f046.png">
向きが違うのはご愛嬌。
このplotは、RGUI上に出ます。


# サンプルデータの生成
えーと、無理にアレする必要はないんですが、まぁ、[これまで](http://qiita.com/kilometer/items/4f48bd50dd47cf6724c7)と合わせますか。

```r
# 乱数のシード
set.seed(11)
# 円のxy座標を整数値で吐き出す。
x <- round(20*cos(seq(0, 2*pi, by=0.05)) + 31)
y <- round(20*sin(seq(0, 2*pi, by=0.05)) + 31)
xy <- cbind(x,y)
# 乱数に縦線と横線。
a <- matrix(runif(61*61, 0, 100/255), 61)
a[31,] <- 150/255; a[,29:31] <- 100/255
# ゴリ押しで円の部分を白抜き
for(i in 1:nrow(xy))  a[xy[i,1], xy[i,2]] <- 1

# imageに変換
dat <- as.Image(a)
```
<img width="300" alt="スクリーンショット 2016-01-28 17.55.08.png" src="md/{EBImage} と {biOps}/8407ef46-7b13-a821-4763-4fda295b51d6.png">

# interporation
解説は、[ここ](http://qiita.com/kilometer/items/4f48bd50dd47cf6724c7)に書いてあります。

{biOps}の**imgScale( )**は、biliner, cubic, splineの3種類が用意されていましたが、
{EBImage}の**resize( )**ではfilter="none" or "bilinear"しか無いっすね。

```r
dat1 <- resize(dat, nrow(dat)*3, ncol(dat)*3, filter="bilinear")
```

<img width="300" src="md/{EBImage} と {biOps}/c7bcf0d5-d0fd-1126-0a9b-4cd823bdf075.png">

"none"だと、そのまま当方拡大(補間しない)。
まぁ、今回は"bilinear"を使います。


次は、Cannyフィルタを{EBImage}で打ってみようか。



