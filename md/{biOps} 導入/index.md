画像解析用Rパッケージ{[biOps](http://ftp.uni-bayreuth.de/math/statlib/R/CRAN/doc/packages/biOps.pdf)}はCRANから消えたんですね。


PC環境：MacBookPro 64bit OS X Yosemite 10.10.5
R環境：R ver.3.1.2(あ！既に古い！)

　

[biOpsのインストール方法](https://code.google.com/p/rimagebook/wiki/biOpsInstallation)を参照。


以下引用。

>    ターミナルで以下のコマンドを実行してライブラリをインストールします．
>
>    $ sudo port install fftw-3 tiff jpeg
>
>    インストールしたライブラリにシンボリックリンクを張ります．
>
    $ cd /usr/include
    $ sudo ln -s /opt/local/include/fftw3.h
    $ for x in /opt/local/include/j*.h; do sudo ln -s $x; done
    $ for x in /opt/local/include/tiff*.h; do sudo ln -s $x; done
    $ cd /usr/lib
    $ for x in /opt/local/lib/libfftw3.*; do sudo ln -s $x; done
    $ for x in /opt/local/lib/libjpeg.*; do sudo ln -s $x; done
    $ for x in /opt/local/lib/libtiff.*; do sudo ln -s $x; done

>    RからdevtoolsパッケージとbiOpsをインストールします．
>
    $ sudo R
    > install.packages("devtools")
    > library(devtools)
    > install_url("http://cran.r-project.org/src/contrib/Archive/biOps/biOps_0.2.2.tar.gz")
    > library(biOps)
>
>以上でbiOpsのインストールは完了です.


