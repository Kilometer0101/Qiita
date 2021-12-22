


![](https://camo.qiitausercontent.com/57ca126d744e52f2623cf6dcdfd1c47e3a11858e/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f38353634346138332d633437372d376365392d363937312d6462353633633164626638632e706e67)

データの畳み込み`nest`を上手く利用して見通しよく処理をしよう、という記事を書きました（[nestしていこう。](https://qiita.com/kilometer/items/e28a7ad381a2ad5dcd68)）。その中で、大した説明もなしにガンガン`map`や`map2`を使っていたのでフォローしておきます。

その前に書いた超長文の中の[繰り返しに関するパート](https://qiita.com/kilometer/items/63f2eab36e268096d313#%E7%B9%B0%E3%82%8A%E8%BF%94%E3%81%97%E3%81%AE%E8%A9%B1)でも`map2`を登場させているので、お前誰だ、というヤツです。

尚、資料としては既に良いものがアレコレあります。

・[そろそろ手を出すpurrr](https://speakerdeck.com/s_uryu/nekosky?slide=26)
・[purrr — ループ処理やapply系関数の決定版](https://heavywatal.github.io/rstats/purrr.html)
・[モダンな繰り返し処理purrrの使い方](https://www.medi-08-data-06.work/entry/how_to_use_purrr1901)

これらの記事とかなり重複する内容ですが、フォローアップという意味で一連の資料をまとめておくために、理解に必要な情報にかなり絞ってまとめました。まずこのレベルで手をつけてみて、慣れてきたら色々と広げていくのが良いかと思います。

# ライブラリ

面倒なので、まとめてattachしておきましょう。

```r
library(tidyverse)
```

# ベクトルの演算

テストのため、単純な独自関数を定義します。

```r
f <- function(dat){ dat + 1 }
```

これをベクトル`vector`に適用すると、

```r
x <- 1:4

x
#> [1] 1 2 3 4

f(x)
#> [1] 2 3 4 5
```

こうなります。
ベクトル演算の場合、個々の要素それぞれにfunctionが適用されています。

ベクトルの取り扱いに関する詳しい情報は、[biostatistics](https://stats.biopapyrus.jp/r/basic/vector.html)さん、[R-Tips](http://cse.naro.affrc.go.jp/takezawa/r-tips/r/14.html)さん等をご参照ください。

入力`x`が、ベクトルを要素に取った`list`だったらどうなるかというと、エラーになります。

```r
x <- list(a = 1:4, b = 3:8)

x
#> $a
#> [1] 1 2 3 4
#> 
#> $b
#> [1] 3 4 5 6 7 8

f(x)
#> x + 1 でエラー:  二項演算子の引数が数値ではありません
```

意図する挙動は、`x$a`に`f`を適用し、`x$b`にも`f`を適用したい。

# forを使う。

`for`文を使ったループで計算するならこうですね。

```r
results <- NULL
for(i in 1:length(x)){
  results[[i]] <- f(x[[i]])
}

results <- set_names(results, names(x))

results
#> $a
#> [1] 2 3 4 5
#> 
#> $b
#> [1] 4 5 6 7 8 9
```

面倒くさいですね。

# mapを使う。


こんな時に、`map`が便利です。

```r
map(x, f)
#> $a
#> [1] 2 3 4 5
#> 
#> $b
#> [1] 4 5 6 7 8 9
```

狙い通りの結果が得られています。
つまり、入力`x`の要素(`.$a`, `.$b`)ごとに関数`f`が適用されています。

ヘルプを表示して関数の詳細を確認してみましょう。

```r
?map
```

|Usage|
|---|
|map(.x, .f, ...)|

|Arguments||
|---|---|
|.x|A list or atomic vector.|
|.f|A function, formula, or vector (not necessarily atomic).|
|...|Additional arguments passed on to the mapped function.|

という事で、入力`.x`はlistかvector、処理に使う関数`.f`が二番目に来ます。

```r
f <- function(dat){ dat + 1 }
map(.x = x, .f = f)    # これを省略してmap(x, f)と書けます。
```

`f`を`map`の中で無名関数として定義することもできます。

```r
map(x, function(dat){ dat + 1 })
```
関数名を考える必要がないという利点があります。
難点は、複雑な処理を入れるとフローを追う(読む)のが大変になります。
トレードオフですね。

更に、チルダ`~`を使ったラムダ式で書くこともできます。

```r
map(x, ~ { . + 1 })
```

[pipe演算子`%>%`](https://qiita.com/kilometer/items/63f2eab36e268096d313#r%E3%81%AE%E3%83%91%E3%82%A4%E3%83%97%E6%BC%94%E7%AE%97%E5%AD%90%E3%81%AE%E8%A9%B1)を使うとこうですね。

```r
x %>% map(~ { . + 1 })
```


こんな事もできます。

```r
x %>% map( ~ { . + 1 } %>% sum)
#> $a
#> [1] 14
#> 
#> $b
#> [1] 39
```

このラムダ式の`.`は、第一引数を参照しています。`.x`と書いても良いです。


デフォルトだと返り値は`list`ですが、wrapper関数を使う事で、出力のデータ型を変える事もできます。よく使う3種類は下記。

```r
x %>% map_df( ~ { . + 1 } %>% sum)  # data.frame形式で出力(tibbleですが)
#> # A tibble: 1 x 2
#>       a     b
#>   <dbl> <dbl>
#> 1    14    39

x %>% map_dbl( ~ { . + 1 } %>% sum)  # 実数ベクトル形式で出力
#>  a  b 
#> 14 39 

x %>% map_chr( ~ { . + 1 } %>% sum)  # 文字列ベクトル形式で出力
#>           a           b 
#> "14.000000" "39.000000" 
```


# 複数の要素を引数にとるなら。

![image.png](https://qiita-image-store.s3.ap-northeast-1.amazonaws.com/0/92401/16bc94ea-1247-e8ab-f8c6-09ad03f0a8f8.png)


`map`は、1つのlistを`.x`に取って、その要素に対して順に`.f`を適用するわけですが、適用したい関数によっては2つの引数を取りたい時もありますね。そんな時には`map2`を使います。

```r
x1 <- list(a = 1:6, b = 3:8)

x2 <- list(a = 6:1, b = 10:15)

map2(x1, x2, ~ .x + .y)
#> $a
#> [1] 7 7 7 7 7 7
#> 
#> $b
#> [1] 13 15 17 19 21 23
```

チルダ式の`.x`は第一引数`x1`を、`.y`は第二引数`x2`をそれぞれ参照します。


3つ以上を参照したい場合は、`pmap`を使います。要素数は揃っている必要があります。

```r
list(1:5, 3:7, 11:15) %>% 
  pmap_dbl(function(x, y, z){x + y * z})
#> [1]  34  50  68  88 110
```

`data.frame`は名前付きlistで要素数が揃っている事が保証されているので相性が良いです。

```r
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap_dbl(function(x, y, z){x + y * z})
#> [1]  34  50  68  88 110
```

### デモpmapハチョットツカイニクイ

例えば、これ↓が通らない。

```r
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap_dbl(function(a, b, c){a + b * c})
#>  .f(x = .l[[1L]][[i]], y = .l[[2L]][[i]], z = .l[[3L]][[i]], ...) でエラー: 
#>   使われていない引数 (x = .l[[1]][[i]], y = .l[[2]][[i]], z = .l[[3]][[i]]) 

# コレ↓は通る
list(1:5, 3:7, 11:15) %>%
  pmap_dbl(function(a, b, c){a + b * c})
#> [1]  34  50  68  88 110
```

pmap内部の`.f`の引数名が、与えられる入力`.l`のnameと一致している必要があるのです。

また、入力は`.l`の一つですので、`.x`や`.y`といった書き方ができず、チルダを用いたラムダ式が書けない。(引っ張りたい要素の数が分からないので当然といえば当然ですが。)

```r
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap_dbl(~{.x + .y * .z})
#>  .f(x = .l[[1L]][[i]], y = .l[[2L]][[i]], z = .l[[3L]][[i]], ...) でエラー: 
#>    オブジェクト '.z' がありません
```

なんともならない。

```r
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap(~{.$x + .$y * .$z})
#>  .$x でエラー: $ operator is invalid for atomic vectors

data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap_dbl(~{.data$x + .data$y * .data$z})
#>  エラー: Result 1 must be a single double, not an integer vector of length 0
#> Call `rlang::last_error()` to see a backtrace
```



どうせ`pmap`を使うような関数は複雑になりそうなので、引数と要素名を合わせた関数を別途定義しておくのが良さそうですね。


```r
f <- function(x, y, z){x + y * z}
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  pmap_dbl(f)
#> [1]  34  50  68  88 110
```

data.frameを取り扱う場合、よっぽどの事がなければ素直に`mutate`で計算してしまうのも手です。

```r
data.frame(x = 1:5, y = 3:7, z = 11:15) %>% 
  mutate(val = x + y * z)
#>   x y  z val
#> 1 1 3 11  34
#> 2 2 4 12  50
#> 3 3 5 13  68
#> 4 4 6 14  88
#> 5 5 7 15 110
```

# nested dataの演算に便利。


![](https://camo.qiitausercontent.com/6e94dace542bde56251d553f6282fc4ea18383a6/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f33383365386365662d656164372d613761662d363738372d3133363337666663343361662e706e67)

```r
dat <- iris %>% 
  group_by(Species) %>% 
  nest

dat
#> A tibble: 3 x 2
#>   Species    data             
#>   <fct>      <list>           
#> 1 setosa     <tibble [50 × 4]>
#> 2 versicolor <tibble [50 × 4]>
#> 3 virginica  <tibble [50 × 4]>
```

nestされている`data`カラムは、listになっています。

```r
dat$data
#> [[1]]
#> # A tibble: 50 x 4
#>    Sepal.Length Sepal.Width Petal.Length Petal.Width
#>           <dbl>       <dbl>        <dbl>       <dbl>
#>  1          5.1         3.5          1.4         0.2
#>  2          4.9         3            1.4         0.2
#>  3          4.7         3.2          1.3         0.2
#> # … with 47 more rows
#> 
#> [[2]]
#> # A tibble: 50 x 4
#>    Sepal.Length Sepal.Width Petal.Length Petal.Width
#>           <dbl>       <dbl>        <dbl>       <dbl>
#>  1          7           3.2          4.7         1.4
#>  2          6.4         3.2          4.5         1.5
#>  3          6.9         3.1          4.9         1.5
#> # … with 47 more rows
#> 
#> [[3]]
#> # A tibble: 50 x 4
#>    Sepal.Length Sepal.Width Petal.Length Petal.Width
#>           <dbl>       <dbl>        <dbl>       <dbl>
#>  1          6.3         3.3          6           2.5
#>  2          5.8         2.7          5.1         1.9
#>  3          7.1         3            5.9         2.1
#> # … with 47 more rows
```


なので、`map`で取り扱うと捗ります。
`mutate`で新たなカラムを作り、そこに`data`の要素を`map`の引数に取り、`.f`を適用した結果を格納します。

![](https://camo.qiitausercontent.com/936c674591fa9c64cfcf752446034dcd7566cb35/68747470733a2f2f71696974612d696d6167652d73746f72652e73332e616d617a6f6e6177732e636f6d2f302f39323430312f38393337613137312d626465302d333266662d616266392d3965616434643966386166662e706e67)

```r
dat %>% 
   mutate(mean = map(data, ~summarise_all(., mean)))
#> # A tibble: 3 x 3
#>   Species    data              mean           
#>   <fct>      <list>            <list>          
#> 1 setosa     <tibble [50 × 4]> <tibble [1 × 4]>
#> 2 versicolor <tibble [50 × 4]> <tibble [1 × 4]>
#> 3 virginica  <tibble [50 × 4]> <tibble [1 × 4]>
```

`$mean`の中身はこんな感じになります。

```r
[[1]]
# A tibble: 1 x 4
  Sepal.Length Sepal.Width Petal.Length Petal.Width
         <dbl>       <dbl>        <dbl>       <dbl>
1         5.01        3.43         1.46       0.246

[[2]]
# A tibble: 1 x 4
  Sepal.Length Sepal.Width Petal.Length Petal.Width
         <dbl>       <dbl>        <dbl>       <dbl>
1         5.94        2.77         4.26        1.33

[[3]]
# A tibble: 1 x 4
  Sepal.Length Sepal.Width Petal.Length Petal.Width
         <dbl>       <dbl>        <dbl>       <dbl>
1         6.59        2.97         5.55        2.03
```

冒頭の繰り返しになりますが、nestを使った畳み込みとmap族を使ったデータ処理については、別途、[nestしていこう。](https://qiita.com/kilometer/items/e28a7ad381a2ad5dcd68)という記事を書いたので、詳細はそちらをご参照ください。

