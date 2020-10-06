---
title: Linux 系OS上で recpt1,epgdump を使って TV番組を録画する録画予約システム
---


## 目的

本プログラム(raspirec) は Linux 系OS上で、
recpt1,epgdump を使って TV番組を録画する録画サーバーを構築するための
録画予約システムです。
<br>
特に、ラズパイのようなシングルボードコンピュータ(SBC)で動作させる事に最適化しています。

なお動作には recpt1 に対応しているドライバーがあるTVチューナー
( アースソフト社製 PT1〜3, PLEX社製 PX-W3U4、PX-W3PE4、PX-Q3U4、PX-Q3PE4 等)
が必要です。

## 特徴

* Raspberry Pi のようなシングルボードコンピュータ(SBC)でも実行できるように、
  機能はシンプルで小型、軽量。
* 処理能力の低いPCでも動作するように最適化。
* 同時録画本数を多くする為に録画中はなるべく負荷を掛けない。
  (raspbery Pi 3B+ で 6本までは確認)
* インストールが容易
* WEBインターフェースで、EPG番組表から録画予約が可能。
* 条件検索を設定することで、自動予約登録が可能。
* 録画したTSファイルを、親機にコピーする機能
* チューナーの数とは別に、同時録画の本数で制限を掛ける事が可能。
  (SBCの場合、能力不足により本数制限が必要になる。)

<!--more-->

## スクリーンショット


| Top画面 <br> [![]({{site.baseurl}}/img/top.png){: .ssimg}]({{site.baseurl}}/img/top.png) | 番組表<br> [![]({{site.baseurl}}/img/prg_tbl.png){: .ssimg}]({{site.baseurl}}/img/prg_tbl.png) | 予約状況表 <br> [![]({{site.baseurl}}/img/rsv_tbl.png){: .ssimg}]({{site.baseurl}}/img/rsv_tbl.png)

| 番組検索 <br>  [![]({{site.baseurl}}/img/search.png){: .ssimg}]({{site.baseurl}}/img/search.png) | mpvモニタ(1) <br> [![]({{site.baseurl}}/img/mpv2.png){: .ssimg}]({{site.baseurl}}/img/mpv2.png)  | mpvモニタ(2) <br> [![]({{site.baseurl}}/img/mpv.png){: .ssimg}]({{site.baseurl}}/img/mpv.png)

 \* mpvモニタ(2)は raspbery Pi 3B+ に接続された PX-Q3U4 の映像を親機のdesktopに表示

| hlsモニタ <br>  [![]({{site.baseurl}}/img/hls.png){: .ssimg}]({{site.baseurl}}/img/hls.png)


## ステータス

* 現在運用試験中。
* とりあえず録画で出来る所まで出来たので、β版で公開開始
* 順次機能追加して行く予定。

## 実行に必要な環境

* Linux系 OSが稼働するPC と OS (ドライバーさえあれば UNIX系なら何でも可)
* ruby  2.5 以上
* sqlite3
* TVチューナー ( recpt1 のドライバーが存在するもの )
* recpt1 ( https://github.com/stz2012/recpt1 を推奨 )
* epgdump ( https://github.com/Piro77/epgdump を推奨 )
* もし b25 デコードするなら b25 ライブラリ + カードリーダー


## 制限事項

* jquery,Materialize を参照しているので、インターネットにアクセス出来る環境
  で動作させる事が必要。
  だたし、あらかじめ参照ファイルをダウンロードして置けばオフラインでも動作
  させる事ができる。(
  [doc/jquery_local.md](https://github.com/kaikoma-soft/raspirec/blob/master/doc/jquery_local.md) を参照 )

* セキュリティにはあまり考慮していないので、インターネット側から
  アクセス出来る状態にしないで下さい。

* android(Fire HD10)版のブラウザでは、一部機能が動作しない。
  ( なぜか firefox と opera では、動作しない箇所が違う。)

## インストール方法 ( Raspbian buster lite の場合 )

* インストールに必要な下記のパッケージを apt install で インストールする。

    * git
    * autoconf
    * raspberrypi-kernel-headers
    * dkms
    * cmake
    * sqlite3
    * ruby
    * ruby-sqlite3
    * ruby-sys-filesystem
    * ruby-net-ssh
    * ruby-sinatra
    * ruby-slim
    * ruby-sass


* recpt1

  ハードに合わせたドライバーをインストールし、
  コマンドラインから実行して録画出来る事を確認しておく。

* epgdump

   epgdump は、同じ名前で仕様の違うものがあるので、
   必ず https://github.com/Piro77/epgdump を使う。

* raspirec ( 基本機能のみ、オプション機能は [別紙]({{site.baseurl}}/src/raspirec-option.html)を参照 )

    1. インストールするディレクトリに移動して

       `% git clone https://github.com/kaikoma-soft/raspirec.git`

       すると raspirec というディレクトリが出来るので、
       それを以下 $BaseDir とする。

    1. 環境に合わせて configファイルをカスタマイズ

       * 雛形の config.rb.sample を $HOME/.config/raspirec/config.rb にコピー
         
       * コピーした config.rb をテキストエディタを使って、
         自分の環境に合わせるように修正する。
         とりあえず最低限必須なのは次のもの。詳細は
         [doc/config.md](https://github.com/kaikoma-soft/raspirec/blob/master/doc/config.md) を参照
         
         ```
         Recpt1_cmd        : recpt1 コマンドの path を指定する。
         Epgdump           : epgdump コマンドの path を指定する。
         BaseDir           : raspirec がインストールされているディレクトリを設定する。 ( raspirec.rb があるディレクトリ )
         DataDir           : データベースや録画したファイルの置き場所を指定する。
         GR_EPG_channel    : 地デジ EPG 受信局を設定する。
         GR_tuner_num      : 地デジチュナー数
         BSCS_tuner_num    : BSCSチュナー数
         ```

       * configファイルを $HOME/.config 以外の任意の場所に置きたい場合は、
         環境変数 RASPIREC_CONF に絶対パスで指定して下さい。

    1. テスト実行

       `% ruby ${BaseDir}/raspirec.rb -f`

       一度フォアグラウンドで実行し、エラーが出ない事を確認する。
       エラーメッセージがでなければ、Ctrl-C で中止する。


* 実行方法

  `% ruby ${BaseDir}/raspirec.rb`

  でデーモンモードで起動する。
  <br>
  すぐに終了するが、バックグラウンドでサービスは走っているので、
  WEBブラウザ で http://ホスト名:4567/ でアクセスする。
  ( ポート番号の 4567はデフォルトの値で、config で変更可)

  なお、OS の boot時に、自動で起動させたい場合は crontab に

  `@reboot /usr/bin/ruby ${BaseDir}/raspirec.rb`

  を記述する。

* 停止方法

  `% ruby ${BaseDir}/raspirec.rb --kill`

  で、デーモンが停止する。
  

## アンインストール方法

  * ディレクトリ BaseDir, DataDir 以下のファイルを削除
  * インストールしたパッケージを apt remove で削除



## 動作確認環境

|              |  その１             | その２
|--------------+---------------------+-----------------------------|
| 機種         |  raspberry pi 3B+   | AMD Ryzen 7 2700 + MEM 16G
| OS           |  Raspbian Stretch   | Ubuntu 18.04.3 LTS
| TVチューナー |  PX-Q3U4            | PT2
| ドライバー   |  px4_drv            | pt1_drv


## リンク

+ [gitHub recpt1]( https://github.com/stz2012/recpt1 ){:target="_blank"}
+ [gitHub epgdump]( https://github.com/Piro77/epgdump ){:target="_blank"}
+ [gitHub px4_drv]( https://github.com/nns779/px4_drv ){:target="_blank"}
+ [PLEX社 Linux用ドライバー]( http://www.plex-net.co.jp/download/ ){:target="_blank"}
+ [ナマケモノの家 raspirec](http://www.asahi-net.or.jp/~sy8y-siy/src/raspirec.html ){:target="_blank"}
+ [gitHub raspirec](https://github.com/kaikoma-soft/raspirec ){:target="_blank"}
+ [gitHub Pages raspirec](https://kaikoma-soft.github.io/raspirec.html ){:target="_blank"}


## 不具合報告

不具合報告等は、[gitHub の issues](https://github.com/kaikoma-soft/raspirec/issues ) の方にお願いします。


## ライセンス
このソフトウェアは、Apache License Version 2.0 ライセンスのも
とで公開します。詳しくは LICENSE を見て下さい。


