# まとめ

![](https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.ap-northeast-1.amazonaws.com%2F0%2F92401%2F1ab4eeba-4440-4c1e-ec8f-7089d40012d2.png?ixlib=rb-1.2.2&auto=compress%2Cformat&gif-q=60&s=240f9965d299275736adc2eab506d963)


group, nest, map, mutateという4つの道具を使ってデータをハンドルする方法を紹介します。[r-wakalang](https://qiita.com/uri/items/5583e91bb5301ed5a4ba)に寄せられる質問の中に、あー、これを知っていれば簡単に出来るのに！というモノが多いので、その際に「この記事読むといいかも！」という内容をまとめました(なのであまりふざけていない)。

![](https://camo.qiitausercontent.com/936c674591fa9c64cfcf752446034dcd7566cb35/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f38393337613137312d626465302d333266662d616266392d3965616434643966386166662e706e67)


レベルとしては[Tokyo.R](https://tokyor.connpass.com/)の[初心者セッション](https://speakerdeck.com/kilometer)のデモンストレーションに使うぐらいのもので、それほど難しくないと思います。細かいテクを省いて、丁寧に追っていけば概念を理解できるように努めました。

【テンプレ冒頭句】
コードはご自分の手元で、コピペではなく、手打ちで実行しながら確かめてみることをお勧めします。

また、発展編についてはいくつか記事を書いているので、そちらもご参照ください。
・mapについてもう少し詳しく：[{purrr} mapを導入しよう。](https://qiita.com/kilometer/items/b4977df268d2c21211fc)
・nestについてもう少し詳しく：[{tidyr} nestしていこう。](https://qiita.com/kilometer/items/e28a7ad381a2ad5dcd68)
・合わせ技一本をもう少し詳しく：[{lme4} 線形混合モデルの取り回し](https://qiita.com/kilometer/items/2c786cca50357c0b9b1e)
・R自体やtidyverseの入門について：[R言語入門（裏口）-- Landscape with R --](https://qiita.com/kilometer/items/63f2eab36e268096d313)

# 準備

```{r}
library(tidyverse)
set.seed(71)
```

`tidyverse`パッケージを読み込んでおく。
乱数を使うので再現性を担保するためにseedを固定する。

# サンプルデータ
```{r}
N <- 15
dat <- tibble(tag1 = sample(LETTERS[1:3], N, replace = TRUE),
              tag2 = sample(letters[1:5], N, replace = TRUE),
              y = rnorm(N),
              x = runif(N))
```

先頭を表示してデータセットを確認してみます。

```{r}
head(dat)

## # A tibble: 6 x 4
##   tag1  tag2        y      x
##   <chr> <chr>   <dbl>  <dbl>
## 1 C     d     -1.15   0.449 
## 2 C     e      0.0100 0.808 
## 3 B     e      2.37   0.239 
## 4 A     a      0.324  0.0138
## 5 C     e     -0.887  0.0279
## 6 B     c     -0.730  0.161
```

#pipeの挙動

パイプ演算子については何度か書いていますので、挙動を紹介するだけに留めます。

<img width=500 src=https://qiita-user-contents.imgix.net/https%3A%2F%2Fqiita-image-store.s3.amazonaws.com%2F0%2F92401%2F94331f51-7b0d-0a1c-757d-048ef71e879f.png?ixlib=rb-1.2.2&auto=compress%2Cformat&gif-q=60&s=eb639557bda70fe9e8c66bd41f2449e5>


#mutateの挙動

<img height="300" src="md/map脳になろう、もしくはnested dataのハンドリング/aed757e2-692d-4604-0241-9895c21cb37b.png">

`mutate()`関数は、カラム編集用の関数です。
基本的には既存のカラムを材料として使って、新たなカラムを作る時に使います。

```{r}
dat %>% 
  mutate(val = 3 * x + 1)

## # A tibble: 15 x 5
##    tag1  tag2        y      x   val
##    <chr> <chr>   <dbl>  <dbl> <dbl>
##  1 C     d     -1.15   0.449   2.35
##  2 C     e      0.0100 0.808   3.42
##  3 B     e      2.37   0.239   1.72
##  4 A     a      0.324  0.0138  1.04
##  5 C     e     -0.887  0.0279  1.08
##  6 B     c     -0.730  0.161   1.48
```

複数のカラムを材料にすることもできます。

```{r}
dat %>% 
  mutate(val = 3 * x + y)
```

関数をかませることもできます。

```{r}
f <- function(A, B){ 3 * A + B }

dat %>% 
  mutate(val = f(x, y))
```

結果は一致するはずなので、試してみてください。

また、カラム名を既存のもので指定すると、結果は上書きされます。

```{r}
dat %>% 
  group_by(tag1) %>% 
  mutate(val = 1)
## # A tibble: 15 x 5
## # Groups:   tag1 [3]
##    tag1  tag2        y      x   val
##    <chr> <chr>   <dbl>  <dbl> <dbl>
##  1 C     d     -1.15   0.449      1
##  2 C     e      0.0100 0.808      1
##  3 B     e      2.37   0.239      1
##  4 A     a      0.324  0.0138     1
##  5 C     e     -0.887  0.0279     1
##  6 B     c     -0.730  0.161      1

dat %>% 
  mutate(val = 1) %>%
  mutate(val = cumsum(val))
## # A tibble: 15 x 5
##    tag1  tag2        y      x   val
##    <chr> <chr>   <dbl>  <dbl> <dbl>
##  1 C     d     -1.15   0.449      1
##  2 C     e      0.0100 0.808      2
##  3 B     e      2.37   0.239      3
##  4 A     a      0.324  0.0138     4
##  5 C     e     -0.887  0.0279     5
##  6 B     c     -0.730  0.161      6
```

#group_byの挙動

`group_by()`関数は、指定されたカラムをカテゴリカル変数として扱い、その水準をグループとしてまとめるという挙動をします。

```{r}
dat %>% 
  group_by(tag1)

## # A tibble: 15 x 4
## # Groups:   tag1 [3]
##    tag1  tag2        y      x
##    <chr> <chr>   <dbl>  <dbl>
##  1 C     d     -1.15   0.449 
##  2 C     e      0.0100 0.808 
##  3 B     e      2.37   0.239 
##  4 A     a      0.324  0.0138
##  5 C     e     -0.887  0.0279
##  6 B     c     -0.730  0.161 
```

一見、何も起こっていませんが、`# Groups:   tag1 [3]`が追加されていますね。
グループとしてまとめるというのはどういうことかというと、次のコードを試してみてください。

```{r}
dat %>% 
  group_by(tag1) %>% 
  mutate(val = 1,
         val = cumsum(val))
```

分かりづらければ、更に`arrange()`関数で並べ替えてみましょう。

```{r}
dat %>% 
  group_by(tag1) %>% 
  mutate(val = 1,
         val = cumsum(val)) %>% 
  arrange(tag1)

## # A tibble: 15 x 5
## # Groups:   tag1 [3]
##    tag1  tag2        y      x   val
##    <chr> <chr>   <dbl>  <dbl> <dbl>
##  1 A     a      0.324  0.0138     1
##  2 A     c     -0.904  0.414      2
##  3 A     d      0.900  0.427      3
##  4 A     d      1.29   0.555      4
##  5 B     e      2.37   0.239      1
##  6 B     c     -0.730  0.161      2
```

group化しなかった時と見比べると、`cumsum()`関数の適用範囲がグループ内に留まっていることがわかります。これは厳密に挙動して、たとえば`lag()`関数のようなものでも使えます。以下を試してみてください。

```{r}
dat %>% 
  group_by(tag1) %>% 
  mutate(val = 1,
         val = cumsum(val)) %>% 
  arrange(tag1) %>% 
  mutate(val = lag(val))
```

group化は、`summarise()`関数と相性が良く、グループ毎の要約統計量を抽出する際にも便利です。

```{r}
dat %>% 
  summarise(x_mean = mean(x),
            y_mean = mean(y))
## # A tibble: 1 x 2
##   x_mean  y_mean
##    <dbl>   <dbl>
## 1  0.411 -0.0386

dat %>% 
  group_by(tag1) %>% 
  summarise(x_mean = mean(x),
            y_mean = mean(y))
## # A tibble: 3 x 3
##   tag1  x_mean y_mean
##   <chr>  <dbl>  <dbl>
## 1 A      0.352  0.402
## 2 B      0.482  0.210
## 3 C      0.390 -0.539
```

複数のカラムを引数に取ることもできます。以下を試してみて下さい。

```{r}
dat %>% 
  group_by(tag1, tag2) %>% 
  summarise(x_mean = mean(x),
            y_mean = mean(y))
```

グループ化することで、水準毎の処理が可能になりました。
ただし、`summarise()`関数での処理が不可逆であることに注意しておいて下さい。


#nestの挙動

![](https://camo.qiitausercontent.com/6e94dace542bde56251d553f6282fc4ea18383a6/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f33383365386365662d656164372d613761662d363738372d3133363337666663343361662e706e67)

`nest()`関数はデータを畳み込む挙動をします。

```{r}
dat %>% 
  nest(.key = "data")

## # A tibble: 1 x 1
##   data             
##   <list>           
## 1 <tibble [15 × 4]>
```

もとの`dat`は`tibble: 15 x 4`でしたが、この結果は`tibble: 1 x 1`になっています。
新しく作られた`data`カラムに1つのobsがあり、`<tibble [15 × 4]>`が`list`として入っています。この新しいカラムはデフォルトでは`data`という名前ですが、変更したいときは、`.key`引数で指定します。

中身を見てみましょう。

```{r}
dat %>% 
  nest() %>% 
  .$data

## [[1]]
## # A tibble: 15 x 4
##    tag1  tag2        y      x
##    <chr> <chr>   <dbl>  <dbl>
##  1 C     d     -1.15   0.449 
##  2 C     e      0.0100 0.808 
##  3 B     e      2.37   0.239 
##  4 A     a      0.324  0.0138
##  5 C     e     -0.887  0.0279
##  6 B     c     -0.730  0.161 
```

`[[1]]`に注意して下さい。

これは`tibble`を要素に取った`list`形式でデータが保持されていて、ということです。
list形式についてはmapの章も参考にして下さい。

事前にgroup化しておくことで、水準に応じてデータを畳み込むことができます。

```{r}
dat %>% 
  group_by(tag1) %>% 
  nest()
## # A tibble: 3 x 2
##   tag1  data            
##   <chr> <list>          
## 1 A     <tibble [4 × 3]>
## 2 B     <tibble [5 × 3]>
## 3 C     <tibble [6 × 3]>

dat %>% 
  group_by(tag1, tag2) %>% 
  nest()
## # A tibble: 9 x 3
##   tag1  tag2  data            
##   <chr> <chr> <list>          
## 1 C     d     <tibble [1 × 2]>
## 2 C     e     <tibble [3 × 2]>
## 3 B     e     <tibble [2 × 2]>
## 4 A     a     <tibble [1 × 2]>
## 5 B     c     <tibble [2 × 2]>
## 6 B     b     <tibble [1 × 2]>
## 7 A     c     <tibble [1 × 2]>
## 8 A     d     <tibble [2 × 2]>
## 9 C     b     <tibble [2 × 2]>
```

この`group_by`からの`nest`を一度に行う`group_nest()`関数も用意されています。

```{r}
dat %>% 
  group_nest(tag1)
```

作られた`data`カラムの中身はこのようになっています。

```{r}
dat %>% 
  group_nest(tag1, tag2) %>% 
  .$data

## [[1]]
## # A tibble: 1 x 2
##       y      x
##   <dbl>  <dbl>
## 1 0.324 0.0138
## 
## [[2]]
```

畳み込まれたデータを広げる場合は`unnest()`関数を使います。
以下を試してみて下さい。

```{r}
dat %>% 
  group_nest(tag1, tag2) %>% 
  unnest()
```

# mapの挙動

![](https://camo.qiitausercontent.com/57ca126d744e52f2623cf6dcdfd1c47e3a11858e/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f38353634346138332d633437372d376365392d363937312d6462353633633164626638632e706e67)

`map()`関数とその一族は、list形式のデータを処理するためのものです。
特に、listの要素それぞれに対して、特定の関数を作用させた結果が欲しい時に使います。


```{r}
my_list <- list(1:2, 3:5)

## [[1]]
## [1] 1 2
## 
## [[2]]
## [1] 3 4 5
```

標的となるlistは`.x`引数に、作用させたい関数は`.f`引数に入れます(引数名は省略可能)。

```{r}
map(.x = my_list, .f = sum)
## [[1]]
## [1] 3
## 
## [[2]]
## [1] 12
```

自分で作った関数を`.f`にとることもできます。

```{r}
f <- function(x){ x + 1 }

my_list %>% 
  map(f)
```

`map()`関数の中で関数を定義することもできます。

```{r}
my_list %>% 
  map(function(x){ x + 1 })
```

作用させる関数をチルダ`~`を使って無名関数として定義することも可能です。

```{r}
my_list %>% 
  map( ~ { . + 1 })
```

このピリオド`.`に、listの要素がそれぞれ代入されることになります。

2つのlistを引数にとる`map2()`関数も用意されています。

```{r}
x1 <- list(a = 1:6, b = 3:8)
x2 <- list(a = 6:1, b = 10:15)

map2(x1, x2, ~ { .x + .y })
## $a
## [1] 7 7 7 7 7 7
## 
## $b
## [1] 13 15 17 19 21 23
```

無名関数の中では、1つめのlistの要素は`.x`、2つめのlistの要素は`.y`にそれぞれ代入されます。当然、入力される2つのlistの要素の数は揃っている必要があります。

`map()`や`map2()`を使うとlistの要素ごとに同一の関数を作用させることができました。
出力もまた、入力と同じ長さのlistであることに注意して下さい。

# group → nest → map → mutate
![](https://camo.qiitausercontent.com/936c674591fa9c64cfcf752446034dcd7566cb35/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f38393337613137312d626465302d333266662d616266392d3965616434643966386166662e706e67)

さて、道具が整いました。

group化しておいてからnestしたデータは、list形式で畳み込まれているわけですが、ここにmapを使って特定の関数を作用させて、その結果をmutateで新しいカラムに格納していきます。

まずは復習。

```{r}
dat_nest <- dat %>% 
  group_nest(tag1) 

## # A tibble: 3 x 2
##   tag1  data            
##   <chr> <list>          
## 1 A     <tibble [4 × 3]>
## 2 B     <tibble [5 × 3]>
## 3 C     <tibble [6 × 3]>
```

この構造を壊さずに、dataの中のxの平均値を計算してみます。

```{r}
dat_nest %>% 
  mutate(x_mean = map(data, ~mean(.$x)))
## # A tibble: 3 x 3
##   tag1  data             x_mean   
##   <chr> <list>           <list>   
## 1 A     <tibble [4 × 3]> <dbl [1]>
## 2 B     <tibble [5 × 3]> <dbl [1]>
## 3 C     <tibble [6 × 3]> <dbl [1]>
```

`mutate()`関数によって作られた新しいカラム`x_mean`には、それぞれ実数(`dbl`)が1つずつ格納されているのがわかります。こんなときは、`map()`関数のwrapperである`map_dbl()`を使うと綺麗にできます。

```{r}
dat_nest %>% 
  mutate(x_mean = map_dbl(data, ~mean(.$x)))
## # A tibble: 3 x 3
##   tag1  data             x_mean
##   <chr> <list>            <dbl>
## 1 A     <tibble [4 × 3]>  0.352
## 2 B     <tibble [5 × 3]>  0.482
## 3 C     <tibble [6 × 3]>  0.390
```

`map_dbl()`関数を使うことで、`x_mean`カラムの形式がlistからdblに変わりましたね。
group_byからのsummariseで`x_mean`を計算した時と比べて、元のデータを保持しているのがポイントです。

こうしておくと、スッキリと統計的検定も可能です。

```{r}
dat_nest %>% 
  mutate(ttest = map(data, ~t.test(.$x, .$y)),
         pval = map_dbl(ttest, ~{ .$p.value }))
## # A tibble: 3 x 4
##   tag1  data             ttest     pval
##   <chr> <list>           <list>   <dbl>
## 1 A     <tibble [4 × 3]> <htest> 0.926 
## 2 B     <tibble [5 × 3]> <htest> 0.745 
## 3 C     <tibble [6 × 3]> <htest> 0.0456
```

部分を取り出して検定することも、見通しよくできます。

```{r}
dat_nest %>% 
  mutate(x = map(data, ~filter(., tag2 != "b") %>% .$x),
         y = map(data, ~filter(., tag2 != "b") %>% .$y),
         ttest = map2(x, y, ~t.test(.x, .y)),
         pval = map_dbl(ttest, ~{ .$p.value }))
## # A tibble: 3 x 6
##   tag1  data             x         y         ttest     pval
##   <chr> <list>           <list>    <list>    <list>   <dbl>
## 1 A     <tibble [4 × 3]> <dbl [4]> <dbl [4]> <htest> 0.926 
## 2 B     <tibble [5 × 3]> <dbl [4]> <dbl [4]> <htest> 0.837 
## 3 C     <tibble [6 × 3]> <dbl [4]> <dbl [4]> <htest> 0.0311
```

このコードが「あーはいはい」と読めれば、あなたは既に**map脳**です。

# Appendix
## 併せて読みたい

冒頭にも書きましたが、それぞれもう少し詳しくは別記事を書いているのであわせてご参照ください。

・mapについてもう少し詳しく：[{purrr} mapを導入しよう。](https://qiita.com/kilometer/items/b4977df268d2c21211fc)
・nestについてもう少し詳しく：[{tidyr} nestしていこう。](https://qiita.com/kilometer/items/e28a7ad381a2ad5dcd68)
・合わせ技一本をもう少し詳しく：[{lme4} 線形混合モデルの取り回し](https://qiita.com/kilometer/items/2c786cca50357c0b9b1e)
・R自体やtidyverseの入門について：[R言語入門（裏口）-- Landscape with R --](https://qiita.com/kilometer/items/63f2eab36e268096d313)


## Enjoy!!

