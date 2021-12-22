
[「データ視覚化のデザイン #1」をmatplotlibで実装する](https://qiita.com/skotaro/items/cdb0732ad1ad2a4b6236)をRのggplot2でやってみるというヤツを[書きました](https://qiita.com/kilometer/items/f3e1d6e900f0be737404)。

別にggplotを使わずともplotでも書けます。
僕が描くなら版(上記事参照)をplotで描いたやつを供養のため残しておきます。

なんというか、1枚図ならggplotを使わなくてもいいんですよね〜。
ぶっちゃけこっちのほうが見通しが良いと感じるのは、古い人間なのかもしれません。
複数の図をまとめて生成する時は積極的にggplotを使います(facet系)。

tidyverseはデータ処理には必須です。

# Fig.1

データは一緒。

```r
dat <- data.frame(name = c("フリーザ", "ギニュー", "クリリン"),
                  val = c(530000, 120000, 10000)) %>% 
  arrange(val)
```

グラフィックデバイスを起動して、描画して、`dev.off()`で閉じる。
この場合は`png`で.pngを出力するデバイスを起動している。

`par`のオプションは、[R-tips](http://cse.naro.affrc.go.jp/takezawa/r-tips/r.html)さんに網羅されているので参照の事。`barplot`では軸を書かないでおいて後から`axis`で色々と設定すると見通しが良い。

```r
png("fig1_plot.png", width = 700, height = 350, pointsize = 22)

par(family = "HiraKakuPro-W3", tcl = -0.2, mgp = c(2, 0.3, 0), mar = c(3, 5, 1, 1))
barplot(dat$val, horiz = T, axes = F, border = "white", space = 0.1)
axis(2, at = c(1:3)-0.5 + 0.1 * (1:3), labels = dat$name, las = T, pos = 0, col = "white")
axis(1, at = seq(0, 5*10^5, length = 3), 
     labels = seq(0, 5*10^5, length = 3) %>% format(big.mark = ",", scientific = F))
mtext("戦闘力", side = 1, at = 5*10^5, line = 1.5)

dev.off()
```

<img src = md/「データ視覚化のデザイン #1」をplotで実装する(in R)/8df9bc65-15ea-5058-35f9-11fce0e41079.png width = 400>


# Fig.2

データ準備は一緒。色系列作っておく。

```r
dat <- data.frame(year = c(2013:2016),
                  Toyota = c(970, 1010, 1015, 1008),
                  VW = c(975, 1020, 1002, 1035),
                  GM = c(975, 985, 995, 999)) 

cols <- c("mediumseagreen", "indianred2", "dodgerblue1")
```

`plot()`を`type="n"`で呼び出して空白を作っておいて、for文で線を追加。
んー図表の外側に文字を書くには`mtext`を使います。

```r
png("fig2_plot.png", width = 700, height = 420, pointsize = 22)

par(tcl = -0.3, mgp = c(2, 0.5, 0), mar = c(3, 4, 2, 4))
plot(0, 0, type = "n", axes = F,
     xlim = c(min(dat$year), max(dat$year)), 
     ylim = c(950, 1050),
     xlab = "", ylab = "Auto sales")
for(k in 2:nrow(dat)){
  lines(dat$year, dat[, k], col = cols[k-1], lwd = 5)
}
axis(1, at = 2013:2016, labels = 2013:2016)
axis(2, at = seq(950,1050, length = 3), labels = seq(950,1050, length = 3))
mtext(colnames(dat)[2:4], side = 4, at = dat[4, 2:4], las = T, col = cols)

dev.off()
```

<img src = md/「データ視覚化のデザイン #1」をplotで実装する(in R)/d85d93cf-d221-1ac9-e522-62bd1707d5cd.png width = 400>



# Fig.3

データは一緒。

```r
library(lubridate)
set.seed(71)

dat <- data.frame(ymd = (ymd("20150501") + months(0:12)),
                  val = seq(450, 990, length = 13) + runif(13, -50, 50)) %>% 
  mutate(time = ymd %>% as.character() %>% str_sub(1, -4))
```

見たまんま...。
`plot`を`type = "o"`で呼び出しているぐらいかな。

```r
png("fig3_plot.png", width = 700, height = 560, pointsize = 22)

par(tcl = -0.2, mgp = c(2.5, 0.3, 0), mar = c(1, 4, 0, 2))
plot(dat$val, axes = F, type = "o", pch = 16,
     xlab = "", ylab = "Monthly active users in million",
     ylim = c(0, 1100))
for(k in seq(1, 13, length = 4)){
  lines(c(k, k), c(0, dat$val[k]), lty = 3)
}
axis(1, pos = 0, mgp = c(2.5, 0.4, 0),
     at = seq(1, 13, length = 4), 
     labels = dat$time[seq(1, 13, length = 4)])
axis(2, pos = 0.2)

dev.off()
```
<img src = md/「データ視覚化のデザイン #1」をplotで実装する(in R)/27a0ee96-24ba-e64e-1b9d-0859a1eccdd7.png width = 400>



