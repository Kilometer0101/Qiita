
[「データ視覚化のデザイン #1」をmatplotlibで実装する](https://qiita.com/skotaro/items/cdb0732ad1ad2a4b6236)をRでやってみるというヤツです。

ggplotではなくplotで描いたバージョンも[公開しました](https://qiita.com/kilometer/items/a4256d979ab6181fadc2)。

# 1. すっきりバープロット

普通に書くとこうなります。

```r
dat <- data.frame(name = c("フリーザ", "ギニュー", "クリリン"),
                  val = c(530000, 120000, 10000))

library(tidyverse)

ggplot(dat, aes(x = name, y = val))+
  geom_bar(stat = "identity")

ggsave("fig1.png", width =5, height = 2.5)
```
<img src = "md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/088f936e-ecb7-c94b-f360-26fb3175ddee.png", width = 400>

まず、メインとなるグラフ形式の整形。
1. X軸並び替え: `aes`の`x`を`reorder(name, val)`で並び替え
2. 数値表示：`geom_text`を使って、3桁切りに直して左揃えで書き込む。
3. XY軸の入れ替え： `coord_flip`。

色を後から調整したいので`col`カラムを作っておいて`color`と`fill`に指定(ひとまずデフォルトの色が入る)。

```r
dat2 <- dat %>% mutate(col = "1")

g <- ggplot(dat2, aes(x = reorder(name, val), y = val, color = col, fill = col))+
  geom_bar(stat = "identity")+
  geom_text(aes(label = format(val, big.mark=",", scientific=F)), 
            adj = 0, nudge_y = 5000)+
  coord_flip()
```
<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/d5166958-334f-faad-25c2-dc06220abbe5.png width = 400>

不要なモノを消します。
あとトウフを避けるためにフォントを指定。

```r
g <- g +
  theme_minimal()+
  theme(text = element_text(family = "HiraKakuPro-W3"),
        axis.text.x = element_blank(),
        axis.title = element_blank(),
        panel.grid = element_blank(),
        legend.position = "none")
```
<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/80394a30-022f-df60-0104-7b3f8c20c9d6.png width = 400>

はみ出しているのでX軸(反転しているのでY軸)範囲を指定。
色を指定して終わり。

```r
g <- g +
  scale_y_continuous(limits = c(0, 605000))+
  scale_fill_manual(values = "dodgerblue1")+
  scale_color_manual(values = "dodgerblue1")

ggsave("fig1.png", g, width =5, height = 2.5)
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/9f1e292a-8086-1fa3-22bd-9e69dca30d20.png width= 400>

# 2. 凡例は使わない

いってみましょう。
普通に書くならこう。
データの縦持ち、横持ちに注意。

```r
dat <- data.frame(year = c(2013:2016),
  Tokyota = c(970, 1010, 1015, 1008),
  VW = c(975, 1020, 1002, 1035),
  GM = c(975, 985, 995, 999))

dat_g <- dat %>% 
  gather(name, val, -c(year))

ggplot(dat_g, aes(year, val, col = name))+
  geom_path()
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/1264e2c4-48c3-e787-a39f-c92daad9c316.png width = 400>


テキスト用のデータを別途準備しておく。

```r
dat_txt <- dat %>%
  filter(year == max(year)) %>% 
  gather(name, value, -c(year))
```

まずは`geom`周り。
1. 横線を引く：`geom_hline`を`geom_path`の前に書く
2. 線を太くする：`size`引数で調整
3. テキストを入れる：`data = dat_txt`を指定する所に注意。


```r
g <- ggplot(dat_g, aes(year, val, col = name))+
  geom_hline(yintercept = 1000, col = "lightgrey")+
  geom_path(size = 1.5)+
  geom_text(data = dat_txt,
    aes(x = year, y = value, label = name),
    adj = 0, size = 5, nudge_x = 0.1)
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/bb26e4cc-0a89-cea5-6215-d30e0b7dda13.png width = 400>


次。`theme`周り。
消すもの消すしてY軸ラベルを追加。
1. `axis.title.y`にて水平書き・垂直中寄せを指定
2. `ylab`にて幅1で折り返しを指定。

```r
g <- g +
  theme_classic()+
  theme(text = element_text(family = "HiraKakuPro-W3"),
        axis.title.x = element_blank(),
        axis.title.y = element_text(angle = 0, vjust = 0.5),
        axis.line.y = element_blank(),
        panel.grid = element_blank(),
        legend.position = "none")+
  ylab(str_wrap("販売台数", width = 1))
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/7bbaf659-80c6-ba36-01e5-0405788fdd46.png width = 400> 

最後。`scale`周り。
んー特にコメントなし。
色はお気に入りのものを[色見本 Rjp-wiki](http://www.okadajp.org/RWiki/?%E8%89%B2%E8%A6%8B%E6%9C%AC)から選びましょう。

```r
g <- g +
  scale_x_continuous(limits = c(2013, 2016.6))+
  scale_y_continuous(breaks = seq(950, 1050, length = 3),
                     limits = c(950, 1050))+
  scale_color_manual(values = c("mediumseagreen", "indianred2", "dodgerblue1"))
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/346052bc-708d-b5f3-7a2a-b8cf67205fef.png width = 400>



一本だけ目立たせる場合、は`scale`周りだけ変えれば良い。

```r
g <- g +
  scale_x_continuous(limits = c(2013, 2016.6))+
  scale_y_continuous(breaks = seq(950, 1050, length = 3),
                     limits = c(950, 1050))+
  scale_color_manual(values = c("chartreuse2", "lightgrey", "lightgrey"))

ggsave("fig2_post4.png", g, width =5, height = 3)
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/f922b42e-703f-ecac-be0a-9962793c9c95.png width = 400>


もちろん、巷で噂の`gghighlight`を使っても良いですよ！
cf. [gghighlightパッケージをCRANで公開しました](https://notchained.hatenablog.com/entry/2017/10/06/114257)

# 3.目盛りラベルは傾けない

いってみましょう。
日付を使いたいので`{lubridate}`を読み込んで、乱数seedを指定。

```r
library(lubridate)
set.seed(71)
```

普通に書くとこんな感じ。

```r
dat <- data.frame(ymd = (ymd("20150501") + months(0:11)),
                  val = seq(450, 990, length = 12) + runif(12, -50, 50)) %>% 
  mutate(time = ymd %>% as.character() %>% str_sub(1, -4))

g <- ggplot(dat, aes(time, val))+
  geom_bar(stat = "identity")

ggsave("fig3_pre.png", g, width = 5, height = 4)
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/9d3f23d2-369d-d284-6e3c-93ee69c78491.png width = 400>


ラベル用のdata.frameを用意する。
データ側にcolカラムをつけておく。(↑の1.と一緒)

```r
dat_txt <- dat %>% 
  mutate(month = month(ymd),
         year = year(ymd))

dat_g <- dat %>% mutate(col = "1")
```

んーしょうがないから、ややこしいことをします。
1. 水平線を書く。問題なし。
2. `dat_g`を使ってヒストグラムを書く。問題なし。
3. `geom_text`で月を書き込む。まーこんなもん。
4. `dat_txt`を加工して年始めに直してから`geom_text`で書き込む。(お作法的には悪)
5. 線分を`geom_segment`で書き込む。
6. x軸は後から書き込む


```r

g <- ggplot()+
  geom_hline(yintercept = c(500, 1000), color = "lightgrey")+
  geom_bar(data = dat_g, 
           aes(time, val, colour = col, fill = col), stat = "identity")+
  geom_text(data = dat_txt,
            aes(x = time, y = 0, label = month), 
            color = "black", vjust = 1, nudge_y = -10)+
  geom_text(data = dat_txt %>% group_by(year) %>% filter(month == min(month)),
            aes(x = time, y = -75, label = year), 
            color = "black", vjust = 1,, nudge_x = 0.2)+
  geom_segment(aes(x = c(0.5, 8.5), y = c(-100, -100),
                   xend = c(0.5, 8.5), yend = c(0, 0)),
               color = "grey50")+
  geom_hline(yintercept = 0, color = "grey50")
```


<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/6248223a-c364-aa77-0fc9-c0d0898c90fb.png width = 400>


消すものを消して、タイトルを縦書きにして、Y軸スケールを合わせる。

```r
g <- g +
  theme_minimal()+
  theme(axis.title.x = element_blank(),
        axis.title.y = element_text(angle = 0, vjust = 0.5),
        axis.text.x = element_blank(), 
        panel.grid = element_blank(),
        legend.position = "none")+
  ylab(str_wrap("M A U (M)", width = 1))+
  scale_y_continuous(breaks = seq(0, 1000, by = 500))
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/ea54a394-a796-3ce7-0738-a0e454eb5470.png width = 400>

# おわり
あれ、終わった。
以下、「自分ならこうする版」を付け加えます。ちょっと批判的になってしまいますが、読みとり手の負荷を最小にするという元記事の意図には全面的にagreeです。あとは、好み＆ケースバイケースなので「これが正解」と主張するものではありません。

## 例えば自分が描くなら：fig1

```r
dat <- data.frame(name = c("フリーザ", "ギニュー", "クリリン"),
                  val = c(530000, 120000, 10000))

ggplot(dat, aes(reorder(name, val), val))+
  geom_bar(stat = "identity")+
  theme_classic()+
  theme(text = element_text(family = "HiraKakuPro-W3"),
        axis.title.x = element_text(hjust = 1),
        axis.title.y = element_blank(),
        axis.line.x = element_line(color = "grey50"),
        axis.line.y = element_blank())+
  ylab("戦闘力")+
  scale_y_continuous(breaks = seq(0, 500000, length = 3),
                     labels = seq(0, 500000, length = 3) %>% 
                       format(big.mark=",", scientific=F),
                     expand = c(0,0))+
  coord_flip()
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/deab3b3b-eae8-c2ea-2f79-b7264878f8ad.png width = 400>

`expand`指定は↑でも使えば良かったですね。。

1. 僕ならこのグラフに色は使わない。資料を通して印象的に使える色の数は非常に限られている(というか限るべき)。青く塗るなら、資料全体で「戦闘力」を青文字にするぐらいの印象づけをする。
2. 僕ならX軸を使う。棒グラフの上に数値が乗るのは分かりやすいようでいて分かりにくい。数値を見せたいのか、比を見せたいのか、ここでは後者なのでは。
3. 僕なら軸の名前はできるだけ図に盛り込む。その方が図単体として意味を持てるので良いと思う。入れないと、「戦闘力比較」という趣旨を持った別の文字列とセットにしないと解釈不能になりかねない。
4. 場合によってはX軸を1000で割る、指数表記にする、など。これは読み手がどういうグラフを読み慣れているかに依存。

## 例えば自分が描くなら：fig2

```r
ggplot(dat_g, aes(year, val, color = name))+
  geom_hline(yintercept = 1000, color = "grey75", linetype = 3)+
  geom_path(size = 2)+
  geom_text(data = dat %>% group_by(name) %>% filter(year == max(year)),
            aes(x = year, y = val, label = name),
            adj = 0, size = 5, nudge_x = 0.1)+
  theme_classic()+
  theme(text = element_text(family = "HiraKakuPro-W3"),
        axis.title.x = element_blank(),
        legend.position = "none")+
  ylab("Auto sale")+
  scale_x_continuous(expand=c(0, 0), limits = c(2012.8, 2016.8))
```
<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/4e592b51-50a6-e1a1-8676-e19a58ab310d.png width = 400>



色については上述の通り。できるだけ使いたくないと思うけれどけれど資料全体の流れの中で、この図で何を主張するかを考えないと決まらない。

1. 僕は1つのグラフに英語・日本語混じりはできるだけ避けるようにしている(Toyotaと描くならAuto saleで良いのでは)。
2. 宙に浮く横線が、僕はあまり好きではない。横線入れるならY軸線入れたくなる。そもそも横線があまり好きではないので、入れるなら超控えめかつ必要最小限にする。
3. 僕はY軸ラベルは素直に書いた方が素直に見える。英語ならこれ一択。この図を提示するコンテキストにおいて「販売台数」についての議論だという事前の誘導を十分にする方が大切だと思います。Y軸ラベルを読み解くのは図を理解した事後なら無理して縦書きにしなくても伝わるような。じゃあなくて良いかというと、例の図単体で意味を持つためには必要。ど〜しても横書きで書きたければグラフタイトルにした方が違和感が少ないかも。あまり日本ガラパゴスな表記を習熟する意味がないという気もする。


<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/93bacdfa-b25e-5913-c476-b2248b435b65.png width = 300>


## 例えば自分が描くなら：fig3

これは...。うーん。。
ちょっと反則してデータ数を12→13にしました。理由は図を見ていただければ。

```r
library(lubridate)
set.seed(71)

dat <- data.frame(ymd = (ymd("20150501") + months(0:12)),
                  val = seq(450, 990, length = 13) + runif(13, -50, 50)) %>% 
  mutate(time = ymd %>% as.character() %>% str_sub(1, -4))

ggplot(dat, aes(ymd, val))+
  geom_segment(data = dat[seq(1, 13, length = 4),],
               aes(x = ymd, y = val, xend = ymd, yend = 0), 
               color = "grey70", linetype = 3)+
  geom_path()+
  geom_point()+
  theme_classic()+
  theme(axis.title.x = element_blank())+
  ylab("Monthly active users in million")+
  scale_y_continuous(expand = c(0,0), limits = c(0, 1100))+
  scale_x_date(breaks = dat$ymd[seq(1, 13, length = 4)],
               labels = dat$time[seq(1, 13, length = 4)])

ggsave("fig3.png", width = 5, height = 4)
```

<img src = md/「データ視覚化のデザイン #1」をggplot2で実装する(in R)/c8203f19-c00a-e20b-600a-76adbbc9d8b0.png width = 400>

業界のお約束が色々あるのでしょうけれど、元図を僕が見ると、各月を独立として扱いたいのか、時系列として扱いたいのか、ちょっと混乱してしまいます。ここでは時系列として、4ヶ月ごとにY=0に垂線を下ろして比っぽく読み取ることもできるようにしました。

軸ラベルについては上に同じです。(MAU(M)の意味はこういう事であってますかね？)
横が良ければタイトルにする、というのも同じです(割愛)。

## おわりのおわりに
正直、思いの外時間がかかりました。こうしたレイアウト調整（僕の周りでは図をお化粧すると表現します）は普段からやっているので楽勝かなと思いきや、日本語の入った図をそもそもホボ作らないのでアレコレぐぐりながらやりました。参照したサイト一覧を作成すれば良かったですね。。

可視化に正解はないけれど、王道はある、と思います。それは図を読む人の負荷を減らす事と、図単体で伝えられる一貫したメッセージを独立して成立させる事、で、これらは実はトレードオフだと思います。どこで止めるかは時と場合とあなた次第、という事で。



----

# 環境

```r
> sessionInfo()
R version 3.4.0 (2017-04-21)
Platform: x86_64-apple-darwin15.6.0 (64-bit)
Running under: macOS  10.13.4

other attached packages:
 [1] bindrcpp_0.2       forcats_0.3.0      stringr_1.3.0      dplyr_0.7.4       
 [5] purrr_0.2.4        readr_1.1.1        tidyr_0.8.0        tibble_1.4.2      
 [9] ggplot2_2.2.1.9000 tidyverse_1.2.1   
```

