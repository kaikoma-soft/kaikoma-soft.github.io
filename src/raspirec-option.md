---
title: raspirec 「hlsモニタ機能」,「mpvモニタ機能」の設定方法
---


## はじめに

ここでは、raspirec のオプション機能である「hlsモニタ機能」,「mpvモニタ機能」
の設定方法について説明する。

なお 「hlsモニタ機能」,「mpvモニタ機能」
は似た機能であるが下記のような相違点がある。

* hlsモニタ機能
  * 利点
    * ブラウザで表示できること
    * 録画中のファイルをモニタ出来ること
  * 欠点
    * 並列動作が出来ない。
    * 立ち上がりが遅い。
* mpvモニタ機能
  * 利点
    * 並列動作が出来る。(PCの処理能力で制限)
  * 欠点
    * X windowの環境が必要


## hlsモニタ機能

HLSモニター機能は、録画中又は放送中の内容をブラウザで閲覧する機能で、
その機能を有効にするには下記の条件を満たす必要がある。

   *  PC に録画と H264等のエンコードを同時に行う処理能力があること。
   *  recpt1 が --b25 オプションが可能なようにビルドされていること。
   *  ffmpeg 等の HLS 変換が出来るソフトウェアがインストールされていること

機能を有効にするには、config.rb 中の下記の定数を設定する。
  1. MonitorFunc を true 
  1. HlsConvCmd に HLS変換スクリプトのパスを設定
  1. StreamDir,MonitorWidth は必要が無ければデフォルトのまま

* config の例
```
MonitorFunc  = true
MonitorWidth = 720
HlsConvCmd   = SrcDir  + "/tool/ts2hls_sample.sh"
StreamDir    = DataDir + "/stream"
```

* Memo
  * HLS変換スクリプトは、サンプルが tool/ts2hls_sample.sh にあるので、
    それを自分の環境に合わせて変更し使用する。
  * config.rb の変更を反映させるには、再起動が必要。
  * モニタは予約された録画が開始される前に自動的に停止する。






## mpvモニタ機能

 mpvモニター機能は、mpv を使ってチューナーの出力を udp
 経由で直接表示する機能で、
 その機能を有効にするには下記の条件を満たす必要がある。

  * PC に mpv による TS 再生を同時に行う処理能力があること。( Raspberry Pi 3B+ では、能力不足)
  *  recpt1 が --b25 オプションが可能なようにビルドされていること。
  * mpv がインストール( apt install mpv )されていて、実行可能なこと( 要 X Window System )
  * チューナーと表示する PC が別マシンの場合は、チューナーから表示PC
    へ、ssh がパスワード無しでアクセス出来るように設定してあること。


機能を有効にするには、config.rb 中の下記の定数を設定する。

  1. MPMonitor を true 
  1. DeviceList_* にデバイスファイル名を設定。
       (デバイスファイル名はドライバによって異なるので、下記は一例)

        |                     | PT2 +  pt1_drv の場合 | PX-Q3U4 + px4_drv の場合 |
        |---------------------|-----------------------|--------------------------|
        | DeviceList_BSCS     | /dev/pt1video0<br> /dev/pt1video1  | /dev/px4video2<br>/dev/px4video3<br>/dev/px4video6<br>/dev/px4video7
        |  DeviceList_GR      | /dev/pt1video2<br>/dev/pt1video3  |/dev/px4video0<br>/dev/px4video1<br>/dev/px4video4<br>/dev/px4video5

  1.  チューナーと表示PCが別のマシンの場合は、
      * RemoteMonitor  を true
      * XServerName に、mpv を実行、表示するマシン名を設定
      * RecHostName に、チューナー(raspirecが実行されている)のマシン名を設定
      * raspirecが実行されているマシンで、下記のコマンドがパスワード無しで実行出来るか確認する。( XServerName の部分は実際のマシン名を)
```
% ssh #{XServerName} env DISPLAY=:0 mpv --idle --force-window=yes
```


  1. チューナーと表示PCが同じのマシンの場合は、
     * RemoteMonitor  を false 
     * XServerName,RecHostName 等は設定不要


* config の例 (チューナーと表示PCが別のマシンの場合)

```
MPMonitor       = true
DeviceList_GR   = %w( /dev/px4video2 /dev/px4video3 /dev/px4video6 /dev/px4video7 )
DeviceList_BSCS = %w( /dev/px4video0 /dev/px4video1 /dev/px4video4 /dev/px4video5 )
MPlayer_cmd     = %w( mpv --deinterlace=yes --autofit=720x405 --quiet )
RemoteMonitor   = true
UDPbasePort     = 12345
XServerName     = "desktop"
RecHostName     = "raspi"
Lsof_cmd        = "/usr/bin/lsof"
```


* Memo
  * config.rb の変更を反映させるには、再起動が必要。
  * モニタは予約された録画が開始される前に自動的に停止する。
