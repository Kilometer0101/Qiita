# やりたいこと
・2D散布図を描画
・プロットの分布を2D正規分布で近似
・プロットの存在確率を示す確率楕円を描く

# 0. サンプルデータ
回転楕円みたいな分布を作る。

```r
set.seed(13)
dat0 <- matrix(rnorm(1000), , 2)
dat0[,1] <- 2 * dat0[,1]

sita <- pi/3
rot <- matrix(c(cos(sita), -sin(sita), sin(sita), cos(sita)),2)

dat <- data.frame(dat0 %*% rot)
```
<img width="176" alt="スクリーンショット 2016-09-02 15.12.56.png" src="md/{ellipse} 確率楕円を描く/985f39f4-eca3-a9cb-3cd6-f6802229232a.png">
# 1. 確率楕円の算出

`ellipse::ellipse`を使って、楕円の中心を`centre`、描きたい確率範囲を`level`で与える。

```r
ellipse(cov(dat), centre=apply(dat, 2, mean), level=0.95)
```

# 2. 描画

### 楕円データの準備

```r
level <- seq(0.5, 0.95, by=0.05)

datE <- NULL
for(i in level){
  datE <-rbind(datE, cbind(i, ellipse(cov(dat), centre=apply(dat, 2, mean), level=i)))
}
datE <- data.frame(datE)
datE$i <- factor(datE$i)
```
<img width="179" alt="スクリーンショット 2016-09-02 15.29.25.png" src="md/{ellipse} 確率楕円を描く/30877111-42aa-c404-ee2b-5af854b24f7d.png">

### 描画

```r
ggplot()+
  layer(
    data = dat,
    mapping = aes(x=X1, y=X2),
    geom = "point",
    stat = "identity",
    position = "identity"
  )+
  layer(
    data = datE,
    mapping = aes(x=X1, y=X2, color=i),
    geom = "path",
    stat = "identity",
    position = "identity"
  )+
  scale_color_manual(values=grey(seq(0, 200, length=10)/255))+
  theme_bw()
```
<img width=550, src=md/{ellipse} 確率楕円を描く/1aad84a5-f8db-6422-e8b6-b632ad1d2968.png>

`layer`を久しぶりに使ったら`position`表記が必須になったようで戸惑った。


### 参考
書いたあとこんなのを見つけて…。
何かパラメータいじったら楕円になりそうな気がしなくもなくもないですね〜。

```r
ggplot(dat, aes(X1, X2))+
  geom_density2d(n=10)+
  geom_point()
```
<img width="550" alt="スクリーンショット 2016-09-02 15.47.24.png" src="md/{ellipse} 確率楕円を描く/b5d56034-1060-ad92-1278-1b40ba8a421b.png">





