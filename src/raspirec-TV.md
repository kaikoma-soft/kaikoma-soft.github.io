---
title: raspirecTV
---


## はじめに

ここでは、
[raspirec](https://github.com/kaikoma-soft/raspirec ){:target="_blank"}
のオプション機能で、Ver1.2.0 から導入された
raspirecTV について説明します。

## raspirecTV とは

raspirecTV とは、独立した GUIで、指定した放送局を recpt1/recdvb
を使ってチューニングし、mpv と連携させ番組を視聴するするものです。
<br>
特徴として、チューナーが接続されたヘッドレスの SBC 上で実行し、
デスクトップ側で mpv を実行し表示する事が出来ます。
<br>
なお、プログラムは独立ですが DB は raspirec のものを参照するので、
raspirec が、正常に稼働している必要があります。


[![main画面 と mpv画面]({{site.baseurl}}/img/TV.png){: .mmimg}]({{site.baseurl}}/img/TV.png "main画面 と mpv画面")

## 実行に必要な環境
* [raspirec](https://github.com/kaikoma-soft/raspirec ){:target="_blank"}
  (Ver1.2.0 以上) 本体が稼働していること。
* mpv 0.32.0 
* ruby-gtk2 3.2.4
* リモート表示する場合
  * リモート側に mpv と socat がインストールされていること。
  * raspirec 側からリモート側のマシンに ssh がパスワード無しで実行出来る事。

## 実行に必要なパッケージのインストール

```
% sudo apt install ruby-gtk2    # raspirec 側
% sudo apt install mpv          # mpv を実行する側
% sudo apt install socat        # mpv を実行する側
```

## 設定

### 通常の場合の config.rb の記述例
<pre>
MPMonitor       = true          # mpv モニタ機能を有効に
#
DevAutoDetection =  true        # 自動検出するか ( true = する )
DeviceList_GR   = []            # 地デジ チューナー デバイスファイル 
DeviceList_BSCS = []            # BS/CS                  〃
DeviceList_GBC  = []            # 三波共用(地デジ/BS/CS) 〃
#
Mpv_cmd     = "/usr/bin/mpv"
Mpv_opt     = %W( --deinterlace=yes --autofit=640x360 --quiet )
#
RemoteMonitor   = <strong>false</strong>         # チューナーと表示が別マシンの場合に ture
UDPbasePort     = 12345         # UDP port 
Lsof_cmd        = "/usr/bin/lsof"
#
#   raspirecTV 
#
Browser_cmd     = "/usr/bin/firefox"
RaspirecTV_font = "Sans 10"
RaspirecTV_GEO  = "50+50"
</pre>

### リモート(表示と、チューナーが別のマシン)の場合の設定

<pre>
MPMonitor       = true          # mpv モニタ機能を有効に
#
DevAutoDetection =  true        # 自動検出するか ( true = する )
DeviceList_GR   = []            # 地デジ チューナー デバイスファイル 
DeviceList_BSCS = []            # BS/CS                  〃
DeviceList_GBC  = []            # 三波共用(地デジ/BS/CS) 〃
#
Mpv_cmd     = "/usr/bin/mpv"
Mpv_opt     = %W( --deinterlace=yes --autofit=640x360 --quiet )
#
RemoteMonitor   = <strong>true</strong>          # チューナーと表示が別マシンの場合に ture
UDPbasePort     = 12345         # UDP port 
Lsof_cmd        = "/usr/bin/lsof"
#
#   raspirecTV 
#
Browser_cmd     = "/usr/bin/firefox"
RaspirecTV_font = "Sans 10"
RaspirecTV_GEO  = "50+50"
#
XServerName     = "desktop"           # mpv を実行するマシン名
RecHostName     = "raspi"             # raspirec,recpt1 を実行するマシン名
RaspirecTV_SOCAT = "/usr/bin/socat"   # RemoteMonitor == true の時のみ
</pre>


## 実行方法
raspirec をインストールしたディレクトリに PATH を通したあと
<pre>
% raspirecTV.rb
</pre>
 

##  Memo
#### フォントの指定方法
* fontconfig のインストール
```
% sudo apt -y install fontconfig
```
* 下記のコマンドで、フォントの一覧から選ぶ。(下記はIPAの例)
```
%  fc-list :lang=ja | grep IPA
/usr/share/fonts/opentype/ipafont-gothic/ipag.ttf: IPAゴシック,IPAGothic:style=Regular
```
* 出力形式は、 "パス付きのファイル名: フォントファミリー名: style=スタイル"
の形式なので、選んだ フォントファミリー名 + フォントサイズ を
config.rb に記述する。
```
RaspirecTV_font = "IPAUIGothic 12"
```
