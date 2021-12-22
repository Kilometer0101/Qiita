{rstan}を使いこなしていたら格好良いですよね。


chainの計算を並列化できたら早くなるな〜と思って調べていたら、メッチャ簡単にできるようじゃないですか！

```
rstan_options(auto_write=TRUE)
options(mc.cores=parallel::detectCores())
```
と2行書くだけ!!
http://heartruptcy.blog.fc2.com/

ところが、自分のRでやってみたら全く並列化しない。
原因は{rstan}のバージョンが2.2な事にあると辺りをつけ更新する事に。

まず、現版を消す。

```
library(rstan)
set_cppo('fast')
detach("package:rstan", unload = TRUE)
remove.packages('rstan')
```

 次。
GuitHubから{rstan}をゲットしよう。
パッケージをGuitHubからRにブッコムには{devtools}が必要。

```
install.packages(“devtools”)
```

次。
最新版rstanは、{StanHeders}, {ggplot2}, {gridExtra}を求めている。
この3つは、CRANにあるけれど、普通にinstall.packages()だと、バイナリ版が優先して入る。ところが、バイナリ版だとバージョンが古くて怒られる。
なので、ソースからコンパイルする。

```
install.packages(“StanHeaders”, type=“source”)
install.packages(“ggplot2”, type=“source”)
install.packages(“gridExtra”, type=“source”)
```

準備完了。
GutHubから{rstan}をブッコム。

```
devtools::install_github(“stan-dev/stan”, subdir=“rstan/rstan”, ref=“develop”)
```
色々エラーが出て不安になるけれど、待っていたら

>DONE(rstan)

と出ればOK。

codeの書き方は変わらず。
並列化の指定は

```
rstan_options(auto_write=TRUE)
options(mc.cores=parallel::detectCores())
```
とする。
並列化した場合、走り出すとエスケープできないので、codeがちゃんと動く事を確認してから並列化にした方がいい。

で、適当なコードを組んで

![スクリーンショット 2015-10-14 19.15.03.png](md/{rstan}を最新版に！＆並列化処理/d156cd38-31d8-46f5-57b5-4707644995a4.png)


おお。

