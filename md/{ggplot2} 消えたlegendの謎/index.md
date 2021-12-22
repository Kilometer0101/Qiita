--2015/1026更新--


みんな大好き$ggplot2$ :notes: 
みんな大好き$iris$ :notes: 

```
> head(iris)
  Sepal.Length Sepal.Width Petal.Length Petal.Width Species
1          5.1         3.5          1.4         0.2  setosa
2          4.9         3.0          1.4         0.2  setosa
3          4.7         3.2          1.3         0.2  setosa
4          4.6         3.1          1.5         0.2  setosa
5          5.0         3.6          1.4         0.2  setosa
6          5.4         3.9          1.7         0.4  setosa
```


# 消えたlegendの謎
例えば、横軸に$Sepal.Length$を取って、縦軸には$Sepal$と$Petal$の2種類の$Width$をプロットしたい。legendも付けたい。

```
ggplot(iris, aes(x=Sepal.Length))+
  geom_point(aes(y = Petal.Width, color="Petal"))+
  geom_point(aes(y = Sepal.Width, color="Speal"))+
  ylab("Width")
```
<img width="350" alt="スクリーンショット 2015-10-23 18.03.03.png" src="md/{ggplot2} 消えたlegendの謎/626e37ad-7662-2c26-52c5-2397e610d509.png">
おぉ、。legendのタイトルは適当に直せそうだけど、
色いじれるのか？これ。

```
ggplot(iris, aes(x=Sepal.Length))+
  geom_point(aes(y = Petal.Width, color="Petal"), color="Blue")+
  geom_point(aes(y = Sepal.Width, color="Speal"), color="Red")+
  ylab("Width")
```
<img width="350" alt="スクリーンショット 2015-10-23 18.04.19.png" src="md/{ggplot2} 消えたlegendの謎/3f281190-fe34-6d89-5681-99c9af06ff05.png">
ん〜〜〜。legendが消えた;;

# もう少し複雑な謎
これを$Species$ごとにグラフを分けたい。

```
ggplot(iris, aes(x=Sepal.Length, color=Species))+
  hogehoge
```
じゃぁダメだろうな〜。
colorの指定がダブルになっちゃう。

```
　　　　　　　　,．-─ ─-､─-､
　　　　　　, イ)ィ　-─ ──- ､ﾐヽ
　　　 　 ノ ／,．-‐'"´ ｀ヾj ii /　 Λ
　　　 ,ｲ／／ ^ヽj(二ﾌ'"´￣｀ヾ､ﾉｲ{
　　 ノ/,／ミ三ﾆｦ´　　　　　　　 ﾞ､ﾉi!
　　{V /ミ三二,ｲ　, -─　 　 　 　 Yｿ
　　ﾚ'/三二彡ｲ　 .:ィこﾗ 　 ;:こﾗ 　j{
　　V;;;::. ;ｦヾ!V　　　 ｰ '′　i ｰ '　ｿ
　　 Vﾆﾐ( 入　､　　 　 　r　　j　　,′
　　　ヾﾐ､｀ゝ　　｀ ｰ--‐'ゞﾆ<‐-イ
　　　　　ヽ　ヽ　　　　 -''ﾆﾆ‐　 /
　 　 　 　 |　　｀､　　　　 ⌒　 ,/
　　　 　 　|　　　 ＞ ---- r‐'´
　　　　　　ヽ＿ 　 　 　 　 |
　　　　　　　　　ヽ ＿ ＿ 」

　　　　　ググレカス [ gugurecus ]
　　　（西暦一世紀前半～没年不明）
```


[ggplot2で同一グラフに2変数の折れ線グラフを描きたい](http://qiita.com/kazutan/items/3b982eb589dcc12cee54)

おお！

```
library(tidyr)
```
このヒトのお世話になります。

```
a <- gather(iris, key=low, value=value, -c(Species, Sepal.Length))
ggplot(a, aes(x=Sepal.Length, y=value, color=low))+
  geom_point()+
  facet_wrap( ~Species, scale="free")
```
![スクリーンショット 2015-10-23 18.19.00.png](md/{ggplot2} 消えたlegendの謎/722d99cc-5771-8eb0-2aae-3ed29238d224.png)


おおー。yes, yes, yes。

色？

色いじりたい？？？

君、色いじりたいの？？？


orz

# 追記(2015/10/26)

[yutannihilation](http://qiita.com/yutannihilation)様より下記コメントをいただき、お勉強です。
自由度が上がってきました。ありがとうございますm(_ _)m

```
ggplot(a, aes(x=Sepal.Length, y=value, color=legend))+
  geom_line(aes(linetype=legend))+
  scale_color_manual(values=c(1,2,4))+
  scale_linetype_manual(values=c(3:1))+
  theme(text = element_text(size=20)) +
  facet_wrap( ~Species, scale="free")
```
<img width="855" alt="スクリーンショット 2015-10-26 10.52.44.png" src="md/{ggplot2} 消えたlegendの謎/fbdcdaf4-d2fa-cc62-09e0-b5526158e1e8.png">

その他、使えそうなオプション、

```
  facet_wrap( ~Species, scale="free_y") # Y軸だけsceleをfree
  theme(legend.position=c(0.7, 0.2)) # 作図画面全体の中での位置
  theme(legend.key.size=unit(2.5, "lines")) # あんま意味分かってない。
     # unitを使うには{grid}が必要
```
とする。


もう、
<img width="654" alt="スクリーンショット 2015-10-26 19.24.18.png" src="md/{ggplot2} 消えたlegendの謎/198b5c17-21df-07c2-e341-da7943e5acb9.png">
やりたい放題

