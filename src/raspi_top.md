---
title: Raspberry Pi 3B+ ＋ PX-Q3U4 で録画サーバー ( TOP )
---



## 背景

現在使っている PT2 + Windows10 のマシンが老朽化しつつあり、
またチューナー数の不足を感じていた時に
Plex社製TVチューナーのドライバーを個人で作成して、
しかもそれが Raspberry Pi で動く、
という情報を見たので、
安価な録画サーバーとして使え無いか試してみました。

ドライバーの作者さんありがとうございます。



## 目的
下記の様な条件の録画サーバーを構築する。

* 必要なチューナー数は BS x 3。
* 24時間運転するので、出来るだけ消費電力が低いもの。
* ACアダプタで FANレス。
 ( PX-Q3U4 は 爆音の FAN 内蔵だが、バラして撤去する。)


## 用意するもの

* Raspberry Pi 3B+ と ACアダプタ
* [Raspbian 4.14.79-v7+](https://www.raspberrypi.org)
* [PX-Q3U4](http://www.plex-net.co.jp/product/px-q3u4/)
* [PX-Q3U4 用のドライバー](https://github.com/nns779/px4_drv)
* SDカード 16G (SDなしHDDのみの構成は不可だった。後述)
* SDカードライター
* 外付け USB HDD (お古の TV用 1T)
* USB カードリーダー
* 作業用のマシン(Linux)



## 結果

* BS x 3 はクリア
* 録画ソフトは、epgrec UNA版を選択。
* 余裕があれば mp4変換や、dlnaサーバー、ファイルサーバー
  をやらせようと考えていたが無理。
* 録画した TSファイルは ftp で、作業PCに転送する。
  ( scp は暗号化の処理が重いので。)
  転送速度は 19.1Mbyte/秒 程度。


## 詳細
1. [OS のインストール]({{site.baseurl}}/src/raspi_OS_install.html )
1. [性能チェック]({{site.baseurl}}/src/raspi_speedchk.html )
1. [試行中間結果]({{site.baseurl}}/src/raspi_result.html )
1. [ドライバ更新結果]({{site.baseurl}}/src/raspi_result2.html )
