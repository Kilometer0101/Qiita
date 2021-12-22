

# 二項分布
基本的な事は、[前記事-β分布-](http://qiita.com/kilometer/items/3780f172bd314e934281)をご参照ください。

二項分布(Binomial distribution)は、コイン投げの結果を表す確率分布でした。


μ: 1回の試行でコインの表が出る確率`0≤μ≤1`
N: 試行回数(自然数)
k: (N回の試行のうち)表が出た回数
として、kの確率分布は、

```math
p(k|N,μ)={}_NC_kμ^k(1-μ)^{(N-k)}\equiv Bin(k|N,μ)
```

このkの期待値は、1回ごとの表確率μとその回数Nの積なので、

```math
\mathbb{E}[Bin(k|N,μ)]=Nμ
```

# 無限の試行

試行回数Nを無限に吹っ飛ばします。

```math
\begin{eqnarray}
\lim_{N \to \infty} Bin(k|N,μ)&=&\lim_{N \to \infty} {}_NC_kμ^k(1-μ)^{(N-k)} \\
\end{eqnarray}
```
を考えるわけですが、この無限回試行の結果表が出る回数の期待値を`λ`と置きます。

```math
\mathbb{E}\big[\lim_{N \to \infty} Bin(k|N,μ)\big]=\lim_{N \to \infty}Nμ=λ
```

さて、無限回数に吹っ飛ばす事を前提に、少し二項分布の式を書き直します。

```math
\begin{eqnarray}
{}_NC_kμ^k(1-μ)^{(N-k)} &=&\frac{N!}{(N-k)!k!}μ^k(1-μ)^{(N-k)} \\
&=& \frac{N(N-1)...(N-k+1)}{k!} μ^k(1-μ)^{(N-k)} \\
&=& \big( N(N-1)...(N-k+1)μ^k \big)  \frac{1}{k!} (1-μ)^{(N-k)} \\
&=& \Big( \frac{N}{N} \frac{(N-1)}{N}...\frac{(N-k+1)}{N}(Nμ)^k \Big)  \frac{1}{k!} (1-μ)^{(N-k)} \\
&=& \Big(1-\frac{0}{N}\Big)\Big(1-\frac{1}{N}\Big)...\Big(1-\frac{k-1}{N} \Big) (Nμ)^k\frac{1}{k!} (1-μ)^{(N-k)} \\
\end{eqnarray}
```

積和部分は、Nを無限に飛ばすと1ですね。

```math
\begin{eqnarray}
\lim_{N \to \infty} Bin(k|N,μ)
&=&\lim_{N \to \infty} \Big(1-\frac{0}{N}\Big)\Big(1-\frac{1}{N}\Big)...\Big(1-\frac{k-1}{N} \Big) (Nμ)^k\frac{1}{k!} (1-μ)^{(N-k)} \\
&=&\lim_{N \to \infty}  (Nμ)^k\frac{1}{k!} (1-μ)^{(N-k)} \\
&=&\frac{λ^k}{k!}\lim_{N \to \infty} (1-μ)^{(N-k)} \\
&=&\frac{λ^k}{k!}\lim_{N \to \infty} \frac{(1-μ)^N}{(1-μ)^k} \\

\end{eqnarray}
```

`λ`の定義より、

```math
μ=\frac{λ}{\lim_{N \to \infty}N}
```
すなわち、

```math
\begin{eqnarray}
\lim_{N \to \infty} Bin(k|N,μ)
&=&\frac{λ^k}{k!}\lim_{N \to \infty} \frac{(1-μ)^N}{(1-μ)^k} \\
&=&\frac{λ^k}{k!}\lim_{N \to \infty} \frac{(1-\frac{λ}{N})^N}{(1-\frac{λ}{N})^k} \\
\end{eqnarray}
```
分母は1になりますね。
また、

```math
e^x=\lim_{n \to \infty}\Big(1+\frac{x}{n}\Big)^n
```
より、

```math
\begin{eqnarray}
\lim_{N \to \infty} Bin(k|N,μ)
&=&\frac{λ^k}{k!}\lim_{N \to \infty} \Big(1-\frac{λ}{N}\Big)^N\\
&=&\frac{λ^k}{k!}e^{-λ} \equiv Po(k|λ)
\end{eqnarray}
```

これをポアソン分布(Poisson distribution)と呼びます。

# Poisson分布とは

これなんぞ、というと、無限回投げた時にλ回の表が出るコインを`hoge`回投げた時に出る表の回数kの確率分布、ですよね。

で、コインの性質を支配する`μ`の性質はどうなってるかというと、
無限回投げてもたったλ回しか表が出ないコインでして、

```math
μ=\frac{λ}{\lim_{N \to \infty}N}
```

ですので、ま、ハッキリ言えば`μ`すなわち「1回の試行で表が出る確率」が**チョー小さい**コインについて考えている、という事です。

# ん？？

問題は、「コインを`hoge`回投げた時に出る表の回数kの確率分布」の`hoge`ってなんぞ、という事です。

これは、特定の期待値を達成する所与のNに対して、`hoge`回という意味です。

ポアソン分布とは、「一定の(空間and/or時間)間隔で発生するイベント」を想定していて、その中で「**稀に生じる**離散的事象」の発生回数の確率分布です、という事。

「稀に生じる」というのがミソで、その稀さ加減：λ=Nμ=1になる「単位N=hoge」が分かっていれば、2N回の試行では「λ=2*hoge」のポアソン分布に従う、と言っているわけです。

例えば100回に1回表が出る事が期待されるコイン(`μ=0.01`)を、1000回投げた時に表が出る回数kの分布は、

```math
p(k|N,μ)=Bin(k|1000,0.01)={}_{1000}C_k(0.01)^k(0.99)^{(1-k)}
```

ですが、μを十分小さいとみなすと、

```math
p(k|λ=Nμ)=Po(k|0.01*1000)=Po(k|λ=10)=\frac{10^k}{k!}e^{-10} 
```

と表現できるという事です。

こちらの方が計算上有利な時があるので、そんな時はポアソン分布を使ってあげましょう。

