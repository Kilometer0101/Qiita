この記事は、58th TokyoRでのLT内容の焼き直しです。

# intro
音の解析って面白そうですよね。

Aさん(ソプラノ)の「あ」とBさん（テノール）の「あ」は、どちらも「あ」として脳に認識されます。それは「い」とは異なる。当たり前に思いますが、では「あ」と「い」の境目に線を引いてくださいと言われると、悩みが大きそうです。例えば、「æ」(aとeが組み合わさった文字)は、日本語の「あ」と「え」の中間的な発音をするわけですよ。catのaの発音ですね。

これは、音の特徴量をどうやって定義するか、という問題に汎化できます。

という事で、Webに転がっている音のサンプルをネタに、音解析のデモを少々。

# dataset
[HeartSound & Murmur Library](http://www.med.umich.edu/lrc/psb_open/html/repo/primer_heartsound/primer_heartsound.html) @ Univ. Michigan
心拍音の公開データです(.mp3形式)。
ダウンロードできます。

取り扱うのは一番上の「Apex Area - Supine, Listening with the bell of stethoscope」にします。心臓の心尖部（僧帽弁領域）の音をbell型聴診器で聞いた音だそうです。5種類の患者さんの音と、対照2名の音があります。

```r
> head(list.files("Michigan_Heart_Sounds"), 7)
[1] "01 Apex, Normal S1 S2, Supine, Bell.mp3"  # Normal
[2] "02 Apex, Split S1, Supine, Bell.mp3"      # Normal
[3] "03 Apex, S4, LLD, Bell.mp3"               # 僧帽弁逸脱症(MVP)
[4] "04 Apex, Mid Sys Click, Supine, Bell.mp3" # 急性僧帽弁逆流症
[5] "05 Apex, S3, LLD, Bell.mp3"               # 冠動脈疾患による僧帽弁逆流症(MR)
[6] "06 Apex, Early Sys Mur, Supine, Bell.mp3" # MVPによるMR
[7] "07 Apex, Mid Sys Mur, Supine, Bell.mp3"   # 古典的MRもしくは心室中隔欠損症
```

では、この8データを分離できるか試してみましょう。

私は心臓に関してはド素人なので、興味のある方はダウンロード先や、下記をご参照ください。

参考：[心音の聴診](http://heart-clinic.jp/index.php?%E5%BF%83%E9%9F%B3%E3%81%AE%E8%81%B4%E8%A8%BA)

# library
今回は、下記を使います。いずれもCRAN版ですので、`install.packages("hoge")`で落とせます。

音を使うパッケージ2つと、何かと捗る`{tidyverse}`。

```r
library("tuneR")
library("seewave")
library("tidyverse")
```

参考：[俺たちのtidyverseはこれからだ！](http://notchained.hatenablog.com/entry/tidyverse)

ちなみに、

```r
> library(tidyverse)

--------------------------------------------
filter(): dplyr, stats
lag():    dplyr, stats
select(): dplyr, MASS
spec():   readr, seewave
```
と出てきますね。
これらの関数を使うときは、明示的に`package名::関数名`とした方が安全です。

# data import

```r
A <- list.files("Michigan_Heart_Sounds")
k <- 1

# read file
dat <- readMP3(paste("Michigan_Heart_Sounds/", A[k], sep=""))

# data structure
str(dat)
> Formal class 'Wave' [package "tuneR"] with 6 slots
  ..@ left     : int [1:2873088] 0 0 0 0 0 0 0 0 0 0 ...
  ..@ right    : num(0) 
  ..@ stereo   : logi FALSE
  ..@ samp.rate: num 44100
  ..@ bit      : num 16
  ..@ pcm      : logi TRUE
```
この音は、stereoじゃなくてmonauralで、そのamplitudeが`@left`に入っていて、16bitデータで、時間分解能は`44100`Hzだという事がわかります。65秒間ぐらいのデータですね。

# 切り出し
データの先頭を切り出して、プロットしてみましょう。

```r
start <- 0 * dat@samp.rate
end <- 1.6 * dat@samp.rate

dat1 <- dat[start:end]

# amplitude
par(mar = c(3, 5, 0.5, 0.5), tcl = -0.2, mgp = c(1.5, 0.3, 0))
plot(dat1)
```
![image.png](md/{tuneR}{seewave} どきどき de R/1c4c6dd7-5dfd-d37e-6e9f-cf60e5b4df3b.png)

```r
# spectrogram
spectro(dat1, 
        collevels = seq(-50, 0, length=20),  # dB range
        palette = reverse.gray.colors.1,     # color palette
        flim = c(0, 1))                      # kHz range
```
![image.png](md/{tuneR}{seewave} どきどき de R/643dc68d-a338-dc3d-0f18-2e2fe5783085.png)

「ドッ」(I音)と「ックン」(II音)が２回繰り返されていますね。

# extract signal
`{seewave}`の`timer`を使って背景ノイズとシグナルを分離します。
このデータはかなりノイズが少ないので楽ですね。
ここではcutoff値を4.5%にしています。

```r
# cutoff: 4.5% of signal
threshold <- 4.5

par(mar = c(3, 3, 0.5, 0.5), tcl = -0.2, mgp = c(1.5, 0.3, 0))
dat_hb <- timer(dat1, threshold = threshold, msmooth = c(50, 0))
```
![image.png](md/{tuneR}{seewave} どきどき de R/e03582fb-595a-e90f-c5a7-62915c9c5f89.png)

長い無音部分は取れていますが、シグナル部分が細く分離していますね。
あと「ドッ」(I音)と「ックン」(II音)が分かれているのでこれは一塊で取り出したい。

```r
> str(dat_hb)
List of 6
 $ s      : num [1:34] 0.04879 0.00454 0.0034 0.00113 0.00113 ...
 $ p      : num [1:35] 0.31319 0.00113 0.00227 0.00454 0.24397 ...
 $ r      : num 0.155
 $ s.start: num [1:34] 0.313 0.363 0.37 0.378 0.623 ...
 $ s.end  : num [1:34] 0.362 0.368 0.373 0.379 0.624 ...
 $ first  : chr "pause"
```

|parameter||
|:-----|:-----|
|s|	duration of signal period(s) in seconds|
|p|	duration of pause period(s) in seconds|
|r|	ratio between the signal and silence periods(s)|
|positions|	a list containing four elements:|
|s.start|	start position(s) of signal period(s)|
|s.end|	end position(s) of signal period(s)|
|first|	whether the first event detected is a pause or a signal|


いろいろ調整すると、無音継続時間が<0.275secぐらいだと、ちょうど「ドックン」が取り出せました。泥臭いやり方です↓。

```r
# ignor first and last pause
dat_hb$p[1] <- dat_hb$p[1]+1
dat_hb$p[length(dat_hb$p)] <- dat_hb$p[length(dat_hb$p)]+1

# ignor pause dulation < 0.275 sec
threshold_pause <- 0.275
a <- c(1:length(dat_hb$p))[dat_hb$p > threshold_pause]

# start and end of signals
list_start <- dat_hb$s.start[a]
list_end <- dat_hb$s.end[a-1]
```

プロットすると、

```r
x <- 1:length(dat1@left)/dat@samp.rate
y <- x

# cutin
for(ii in 1:length(list_end)){
  y[y >= list_start[ii] & y <= list_end[ii]] <- 100
}
# cutoff
y[y < 100] <- 0

# plot
par(mar = c(3, 3, 0.5, 0.5), tcl = -0.2, mgp = c(1.5, 0.3, 0))
plot(x, y, type = "l", col = "Red", yaxt = "n")
par(new = T)
plot(dat1)
```
![image.png](md/{tuneR}{seewave} どきどき de R/6c57841b-9af1-5b2e-42e3-0f187a9d26ef.png)

このシグナル部分を別の.wavにoutputしておきましょう。

```r
i <- 1
dat_i <- dat[list_start[i]:list_end[i]]
writeWave(dat_i, paste("hoge", i, ".wav", sep =""))
```

7種類のデータからそれぞれ「ドックン」を取り出すとこんな感じ。
（全て同じ値のthresholdで取り出せます。）
![a.png](md/{tuneR}{seewave} どきどき de R/5d7a3b4d-f5a9-32cd-8740-8ae204d627fa.png)



#  statistical properties
`{seewave}`の`specprop`を使って音の特徴量を算出しましょう。
これを全部の「ドックン」について実行して、結果を集計します。

```r
prop_dat <- specprop(seewave::spec(dat_i, f = dat_i@samp.rate, plot =F))

str(prop_dat)
> List of 14
 $ mean    : num 336
 $ sd      : num 577
 $ median  : num 321
 $ sem     : num 5.99
 $ mode    : num 61.8
 $ Q25     : num 200
 $ Q75     : num 411
 $ IQR     : num 212
 $ cent    : num 336
 $ skewness: num 9.41
 $ kurtosis: num 108
 $ sfm     : num 0.0028
 $ sh      : num 0.595
 $ prec    : Named num 2.38
  ..- attr(*, "names")= chr "x"
```

|parameters||
|:---|:---| 	
|mean| 	mean frequency (see mean)|
|sd| 	standard deviation of the mean (see sd)|
|sem| 	standard error of the mean|
|median| 	median frequency (see median)|
|mode| 	mode frequency, i.e. the dominant frequency|
|Q25| 	first quartile (see quantile)|
|Q75| 	third quartile (see quantile)|
|IQR| 	interquartile range (see IQR)|
|cent| 	centroid, see note|
|skewness| 	skewness, a measure of asymmetry, see note|
|kurtosis| 	kurtosis, a measure of peakedness, see note|
|sfm| 	spectral flatness measure (see sfm)|
|sh| 	spectral entropy (see sh)|
|prec| 	frequency precision of the spectrum|

# multivariate analysis
集計するとこんな感じになるはずです(以後の計算に使ったパラメータだけ表示)。
1rowが1ドックンですね。

```r
> head(round(dat_all,3))
  datID    mean       sd   mode kurtosis   sfm    sh dulation
1     1 310.228 1773.836 77.604 1023.705 0.006 0.428    0.387
2     1 313.497 1788.627 77.833 1014.725 0.007 0.425    0.386
3     1 304.034 1750.256 77.833  997.389 0.007 0.425    0.386
4     1 306.773 1761.094 78.062  988.200 0.008 0.426    0.384
5     1 313.893 1791.810 77.604 1025.223 0.007 0.427    0.387
6     1 305.961 1761.365 77.604 1012.312 0.006 0.426    0.387
```

ここまできたら、もうお好きにどうぞ、ですね。

`prcomp`でPCAをかけてもいいし、`isoMDS`でMDSをかけてもいいですね。
方法論とRでの実装はここでは説明しません。
資料は色々ありますし、教科書ですと[多次元データ解析法](https://www.amazon.co.jp/dp/4320019229)がお気に入りです。

<img, height=400, src=md/{tuneR}{seewave} どきどき de R/98d8e69f-d06b-77d8-48d2-2ca1796354a3.png>

どちらでも良い分離が得られていますね。
NormalのID1, ID2が接近しているのは納得ですね。


## appendix

```r
> sessionInfo()
R version 3.3.1 (2016-06-21)
Platform: x86_64-apple-darwin13.4.0 (64-bit)
Running under: OS X 10.10.5 (Yosemite)

locale:
[1] ja_JP.UTF-8/ja_JP.UTF-8/ja_JP.UTF-8/C/ja_JP.UTF-8/ja_JP.UTF-8

attached base packages:
[1] stats     graphics  grDevices utils     datasets  methods   base     

other attached packages:
 [1] purrr_0.2.2     readr_1.0.0     tidyr_0.6.0     tibble_1.2     
 [5] tidyverse_1.0.0 ggplot2_2.2.1   dplyr_0.5.0     tuneR_1.3.1    
 [9] seewave_2.0.5   MASS_7.3-45    

```

