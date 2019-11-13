---
layout: post
title: CMcut4U ： Linux(Unix 系 OS) 上で FFmpeg,OpenCV を使って自動 CMカット
category: software
excerpt_separator:  <!--more-->
comments: false
tags: linux ruby opencv 自動CMカット ffmpeg
description: Linux(Unix 系 OS) 上で ffmpeg,OpenCV 等を使って自動 CMカット
---

-----------------
## 背景

従来 Unix系OS 上で 自動CMカットを行おうとすると comskip
ぐらいしか選択肢が無かった。
しかし comskip は精度が悪かったり、
誤判定した時のリカバリが面倒だったりしたので、
自作することにしました。

-----------------
## 目的

本プログラムは、「PT2等で録画した mpeg2tsファイルを H.265(HEVC) にエンコードしつつ同時に自動で CMカットを Linux(Unix 系 OS) 上で実行する」為のものです。

-----------------
## 特徴

* ロゴファイルを用意しておけば、条件が良ければほぼ全自動で
  CM カットして H.265(HEVC) にエンコード出来る。
* 分割したチャプター数、時間を期待値と比較して結果を表示。
* 期待値と異なった場合は、GUI で修正パラメータを入力してリトライ出来る。
* デメリット
  * 最終確認は人間が目視で行う必要がある。
  * カットの精度はフレーム単位ではなく秒単位であり、
    またマージンを取っているので、CM部分が多少残る。

<!--more-->

-----------------
## 処理概要

CMカット単体の処理の概要は下記の通りです。

* 入力は PT2 で録画した mpeg2ts ファイル
* TSファイルを ffmpeg で、音声(wav)とスクリンーショット(jpeg)に分解。
* 音声ファイルを解析して、無音部分でチャプター分け。
* Opencv のテンプレートマッチを使って、jpeg にロゴマークがあるかで、
  本編／CM  を判定 (あらかじめロゴファイルは手作業で抽出しておく)
* 経験則で、チャプターを整理統合する。
* 得られたチャプター情報を元に TS をチャプター毎に mp4 にエンコード
* 最後にエンコードした mp4 ファイルを、連結して本編のみの mp4 ファイルを出力する。

これを、TSdir で指定したディレクトリ以下の .ts ファイルに対して繰り返し、
Outdir で指定したディレクトリに mp4 ファイルを出力します。


-----------------
## 実行に必要な環境

* Ubuntu 18.04.1 LTS (多分Unix系ならなんでも)
* ruby  2.5.1p57
* ruby-gtk2 3.2.4
* ruby-wave
* python 2.7.15rc1
* Opencv 3.4.1
* ffmpeg(ffprobe) 4.0
* mpv 0.27.2
* gimp ver.2.8.22
  (logoファイルの抽出に使用。矩形領域の切り抜きができれば何でも可)
* X window (fixGUI.rb の実行に必要)


-----------------
## インストール

下記のコマンド例は、Ubuntu でのものです。
他の OS の場合は、コマンドやパッケージ名は適宜変更して下さい。

### ruby
```sh
$ sudo apt install ruby
$ sudo apt install ruby-gtk2
$ sudo gem install wav-file
```

### python
```sh
$ sudo apt install python-dev python-numpy
```

### ffmpeg & mpv
```sh
$ sudo apt install ffmpeg
$ sudo apt install mpv
```

### opencv  
opencv は公式パッケージには無いので、
[opencvの 公式ページ](https://docs.opencv.org/master/d7/d9f/tutorial_linux_install.html)
の手順に従ってソースからインストール。
```sh
$ sudo apt install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
$ sudo apt install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
$ mkdir tmp ; cd tmp
$ git clone https://github.com/opencv/opencv.git
$ git clone https://github.com/opencv/opencv_contrib.git
$ cd opencv ; mkdir build ; cd build
$ cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..
$ make
$ make install
```


### 本ソフト

1. インストールするディレクトリを決める。( 例:~/video/CMcut4U  )
1. そこに git-hub から
    <https://github.com/kaikoma-soft/CMcut4U/archive/master.zip>
   をダウンロードして展開する。
   または
    ```sh
    git clone https://github.com/kaikoma-soft/CMcut4U.git .
    ```
    を実行する。

1. 環境変数 PATH に上記のディレクトリを追加する。
1. const.rb の中身を自分の環境に合わせて、書き換える

 | パラメータ名   | 意味                                   |
 |-------------|---------------------------------------|
 | Top         | TSファイル等のデータを格納するディレクトリ |
 | CPU_core    | CPUのCPUのコア数                        |
 | $ffmpeg_bin | ffmpeg の実行ファイル名                    |
 | $nomalSize  | エンコード後の本編画面サイズ               |
 | $comSize    | エンコード後の CM 画面サイズ               |
 | $python_bin | python の実行ファイル名                    |

1. 入力ファイル、出力ファイル、作業用ディレクトリを作成する。  
   ( 例は、Top がそのままの場合 )
    ```sh
    % mkdir $HOME/video
    % cd $HOME/video
    % mkdir TS mp4 logo work
    ```

-----------------
## ディレクトリ構造

### 入力データ( mpeg2ts ファイル )

入力の TS ファイルは、TS ディレクトリ以下に番組単位のサブディレクトリを
作りその中に置く。
```
Top
├── TS
│   ├── 番組名-1
│   │   ├── タイトル #01.ts
│   │   ├── タイトル #02.ts
│   │   └── ...
│   ├── 番組名-2
│   │   ├── タイトル #01.ts
│   │   ├── タイトル #02.ts
│   │   └── ...
│   └── ...
│       ├── ...
│       ├── ...
│       └── ...

```

### 出力ファイル(mp4ファイル)

  出力の mp4 ファイルは  Top/mp4 以下に TSディレクトリと相似したものが
  生成される。


-----------------
## 使用方法

1. エンコード対象の TS ファイルを「TS/番組名」ディレクトリの下に置く。
1. cmcuterAll.rb を実行する。   
   この段階では、logoファイルが存在しないので、スクリーンショットと
    wav ファイルを作業ディレクトリに作っただけで、エラーで止まる。
1. logoファイルを作成する。
   ( [詳細はこちら]({{site.baseurl}}/create-logo/) を参照して下さい。)
    1. スクリーンショットが保存されたディレクトリを指定して
      logoAnalysisSub.py を実行する。
    ```sh
    % logoAnalysisSub.py --dir work/番組名/タイトル/SS
    ```

    1. スクリーンショット画像が表示されるので、
       ロゴマークが明瞭な時点の画像を保存する。

         | キー      | 意味                                       |
         |-----------|--------------------------------------------|
         | j         | 60コマ( 30秒)飛ぶ                          |
         | k         | 60コマ( 30秒)戻る                          |
         | s         | 現在の画像を保存する。                     |
         | b         | 前のコマ( 0.5秒)に戻る                     |
         | q         | 終了                                       |
         | 上記以外  | 次のコマ( 0.5秒)に進む                     |
    1. 保存した画像の下部（白黒＋強調）部分のロゴマークを
        を gimp を使って、最小限の大きさで矩形領域の切り抜き
        (「ツール」-> 「変形ツール」-> 「切り抜き」)をする。
    1. 「画像」-> 「モード」-> 「グレイスケール」で、グレイスケール化する。
    1. logo ディレクトリの下に、画像を保存する。
1. logo-table.yaml を書き換える。  
   TS ディレクトリの下に logo-table.yaml が自動作成されているので、
   その中の logofn パラメータを、上で保存したlogo ファイル名に書き換える。
   ```
      XXXXX:
          :logofn: YYYY.png
          :cmlogofn:
          :position: top-right
          :chapNum: 6
          :duration: 465
   ```
1. 再度 cmcuterAll.rb を実行する。  



-----------------
## 実行コマンドの説明

+ cmcuterAll.rb
    - 機能  
      TS ディレクトリの tsファイルに対して解析を行い、mp4(HEVC) エンコードを行う。
    - オプション

         | option | 意味                                       |
         |--------|--------------------------------------------|
         | ```-f```     | lock を無視して実行する。                  |
         | ```--co```   | CMカットに必要な計算のみ行う。mp4(HEVC)化のエンコードは行わない |
         | ```--cm```   | 本編画面サイズを $comSize にする。 |
         | ```--ng```   | NG なものだけ対象にする。(cmcuterChk.rb 参照)             |
         | ```--ic```   | chapList.txt のハッシュチェックを行わない。(chapList.txt参照)|
         | ```--uc```   | 中間結果ファイルを再利用する。|
         | ```--sa n``` | 誤差の許容範囲を n 秒にする。デフォルトは 3秒 |
         | ```--dd n``` |  n = 1 : 出力 mp4 ファイルを削除                          |
         |              |  n = 2 : 上記 + chapList ファイルを削除                   |
         |              |  n = 3 : 上記 + 作業ディレクトリの logo 解析キャッシュを削除    |
         |              |  n = 4 : 上記 + 作業ディレクトリのチャプタ毎 mp4 ファイルを削除 |
         |              |  n = 5 : 上記 + 作業ディレクトリの wav,jpeg ファイルを削除      |
         | ```--ar```   | 作業用ファイルの自動削除                                        |


+ cmcuterChk.rb
    - 機能  
      CMカットが正しく行われたかチェックする。  
      logo-table.yaml ファイルに書かれた値と比較して、OK/NG を表示する。
      チェックするのは、mp4ファイルがあるか、チャプター数、本編の時間。

    - オプション

         | option       | 意味                                          |
         |--------------|-----------------------------------------------|
         | ```--ng```   | 結果が NG な番組だけ表示する。                |
         | ```--sa n``` | 誤差の許容範囲を n 秒にする。デフォルトは 3秒 |


    - 出力例
         ```
         >>>>>  番組名 XXXXX  <<<<<<<
          ( chapNum=10, duration=1440 )

         No       Name    mp4 chap   duration      result
          1         01     ○   10     1440.0         OK
          2         02     ○   10     1440.0         OK
          3         03     ○   10     1440.0         OK
          4         04     ○   10     1440.0         OK
          5         05     ○   10     1440.0         OK
          6         06     ○   10     1440.0         OK
          7         07     ○   10     1440.0         OK
          8         08     ○   10     1440.0         OK
          9         09     ○   10     1440.0         OK
         10         10     ○   10     1440.0         OK
         11         11     ○   10     1440.0         OK
         12         12     ○   10     1430.0(-10)    NG
         ```


+ fixGUI.rb
    - 機能  
      修正ファイル( Top/TS/番組名/fix.yaml ) を GUIで作成する。
      これにより、自動判定が間違っている場合に、強制的に 本編/CM
      を固定することが出来る。  

      使い方は、
      + 対象のTSファイルを指定する。
      + 「計算」ボタンを押す。
      + 「mpv 起動/終了」ボタンを押すと mpv が起動する。
          mpv が起動した状態で、表中に時間の欄をマウスでクリックすると、
         mpv がその時間まで seek するので、自動判定の結果が正しいか目視で
         確認する。
      + 誤判定が見つかったの行の fix の欄をマウスでクリックし、
        本編、CM  の固定を選択する。
      + 再度「計算」ボタンを押し、画面下部の結果表示が○になっている事を
        確認して「終了」する。

       ![GUI画面の例]({{site.baseurl}}/img/fixGUI.png)


    - オプション

         | option       | 意味                                           |
         |--------------|------------------------------------------------|
         | ```--uc```   | 中間結果ファイルを再利用する。|
         | ```--sa n``` | 誤差の許容範囲を n 秒にする。デフォルトは 3秒 |


    - 注意  
      修正した mp4 を得るには cmcuterAll.rb を実行する必要がある。


+ logoTblEdit.rb
    - 機能  
      修正ファイル( Top/TS/logo-table.yaml ) を GUIで編集する。

      使い方は、
      + 対象のディレクトリを選択する。
      + パラメータを入力する。
      + 「保存」ボタンを押し、値をファイルに書き込む。
      + 「閉じる」ボタンを押し、終了する。


       ![GUI画面の例]({{site.baseurl}}/img/logoTblEdit.png)


    - オプション

       なし




+ logoAnalysisSub.py
    - 機能  
      スクリーンショットの画像と logoファイルのテンプレートマッチングを行う。
      ```--dir``` オプションだけ指定された場合は、表示モードになる。

    - オプション

         | option            | 意味                                           |
         |-------------------|------------------------------------------------|
         | ```--dir name```  | スクリーンショットが保存された dir を指定する。|
         | ```--logo name``` | logo ファイル名を指定する。                    |


-----------------
## 設定ファイルの説明

+ logo-table.yaml

  + 番組名単位で、下記のパラメタを設定するファイル。
    エントリーはディレクトリの有無で、自動的に作成／削除されるので、
    値を書き換える。( Top/TS/logo-table.yaml )

    | keyword   | 意味                                            |
    |-----------|-------------------------------------------------|
    |:logofn:   | 本編を認識する為の logo ファイル名。( Top/logo 以下の相対パス)|
    |:cmlogofn: | CM 判定の為の logo ファイル名（[詳細は後述]({{site.baseurl}}/create-logo/#cm-ロゴファイルの作成)）     |
    |:position: | logo の画面上の位置。デフォルトは "top-right" で "top-left", "bottom-left","bottom-right" が指定可  |
    |:chapNum:  | チャプター数                                    |
    |:duration: | 本編の時間(秒数)                                |
    |:monolingual| 二カ国放送を日本語のみにするオプションを追加。デフォルト 0 <br>   0 : なし <br> 1 : ```-map_channel 0.1.0  -map_channel 0.1.0``` (音声の左右の場合)<br> 2 : ```-ac 1 -map 0:v -map 0:1 ``` (ストリームが別の場合)|
    |:audio_only| logo解析を行わず、音声のみでチャプター割を行う。true/false デフォルトは false |
    |:fade_inout| フェードイン、アウトを行う true/false。 デフォルトは true |
    |:ffmpeg_vfopt | mp4エンコード時の -vf オプションに任意の文字列を追加する。(4:3に変換する時など) |
    |:ignore_endcard |  Endcard の検出の無効化 true/false デフォルトは false |
    |:ignore_check | このディレクトリ cmcuterChk の対象外とする true/false デフォルトは false |
    |:mp4skip      | このディレクトリは無視する true/false デフォルトは false  |
    |:cmcut_skip   | CMカット処理は行わず、丸ごと mp4化する。true/false デフォルトは false  |
    |:opening_delay | 本編の開始時間を指定した秒数遅延させる。デフォルト 0 |
    |:closeing_delay | 本編の終了時間を指定した秒数遅延させる。デフォルト 0 |
    |:nhk_type     | 本編途中にCMが無く、本編の前後に長い無音期間が有る場合 true。デフォルトは false  |


  + chapNum と duration は、末尾に数字 0〜9 を付けて複数記述可
  + 記述例  
      ```
      XXXXX:
          :logofn: YYYY.png
          :cmlogofn:
          :position: top-right
          :chapNum: 6
          :chapNum1: 8
          :duration: 465
          :duration2: 495
          :monolingual: 2
          :audio_only: true
          :fade_inout: false
          :ffmpeg_vfopt: crop=1440:1080:240:0
      ```

+ fix2.yaml
  + 自動判定が間違っている場合に、強制的に 本編/CM を固定する修正情報ファイル。
    + パスは Top/TS/番組名/fix2.yaml
    + Ver 0.7.0 以前は ファイル名は fix.yaml
      書式は同じだが解釈に互換性はない
  + 記述例  

      ```yaml
      ---
      - - All
        - 1606
        - HonPen
      ```

+ chapList.txt
   + チャプター割の計算結果が格納されたファイル。
     ( $Top/work/番組名/タイトル/chapList.txt)  
     計算結果が間違っていた場合、このファイルを修正して再実行すると
     それに従って、再変換される。     
     なお修正した場合は cmcutAll.rb 実行時に --ic オプションを付けて実行する必要がある。
    （付けないと自動判定の計算が行われてしまう。）

  + 例  

      ```
      0:00:00.00  CM
      0:01:36.90  HonPen
      0:04:11.92  CM
      0:06:27.37  HonPen
      0:15:07.89  CM
      0:17:22.88  HonPen
      0:29:06.88  CM
      0:30:06.00  EOF
      ```

-----------------
## 特殊ファイルの説明

+ cmcut.skip

  TS/番組名/cmcut.skip のファイルが存在すると、そのディレクトリにある TSファイルは、
  CMcut の計算処理を行わず、丸ごと H.265(HEVC) にエンコードを行う。

+ mp4.skip

  TS/番組名/mp4.skip のファイルが存在すると、そのディレクトリにある TSファイルは、
  全て無視される。


-----------------
## カスタマイズ

cmcuterAll.rb と同じ場所に、override.rb という名前でファイルを置くと、
メソッドのオーバーライドが可能です。  
具体的にはハードウエアエンコードを使用したい場合などに用います。

+ override.rb の例
```
  class Ffmpeg
    #
    #  h264_nvenc を使用
    #
    def ts2x265( opt, debug = false )
      arg = %W( -y )
      arg += %W( -ss #{opt[:ss]} -t #{opt[:t]} ) if opt[:ss] != nil
      arg += %W( -i #{@tsfn} )
      arg += %W( -vcodec h264_nvenc -preset fast -acodec aac )
      arg += %W( -s 640x360 #{opt[:outfn]} )
      system2( "/usr/bin/ffmpeg", *arg )
    end
  end
```


-----------------
## 性能

+ 環境
    + Core i5-3570 CPU @ 3.40GHz
    + Mem 8G
    + Ubuntu 18.04.1 LTS

+ ソース
    + BS11 アニメ 30分
    + 画面サイス 1920x1080
    + ファイルサイズ 4,232,185,312 Byte

+ 出力ファイル
  + 画面サイス       1280x720         
  + ファイルサイズ   111,292,555 Byte

+ 実行時間

   | 項目                   | 時間(秒) |
   |:-----------------------|---------:|
   | スクリーンショット作成 | 92.17    |
   | wav 作成               | 20.64    |
   | logo 解析              | 30.43    |
   | チャプター分割         |  1.95    |
   | HEVCエンコード         | 1684.95  |
   | 計                     | 1830.14  |


+ CM判定実績( 2018年夏アニメ )

   |タイトル                |放送局 | 話数 |fix数| fix内容                         | 結果|
   |------------------------|-------|------|-----|---------------------------------|-----|
   |3D彼女 リアルガール     | BS日テレ|12話|全1+1| 前番組の最後と予告編を誤判定    | △  |
   |あそびあそばせ          | BS11  | 12話 | 0   |  -                              | ○  |
   |すのはら荘の管理人さん  | BS11  | 12話 | 1   | 12話 提供+EndCard をCM と誤判定 | △  |
   |はたらく細胞            | BS11  | 13話 | 0   |  -                              | ○  |
   |はねバド!               | BS11  | 13話 | 0   |  -                              | ○  |
   |ゆらぎ荘の幽奈さん      | BS11  | 12話 | 0   |  -                              | ○  |
   |アンゴルモア元寇合戦記  | BS11  | 12話 | 0   |  -                              | ○  |
   |オーバーロードⅢ        | BS11  | 13話 | 0   |  -                              | ○  |
   |ハイスコアガール        | BS11  | 12話 | 1   | 11話最後のおまけ15秒部分        | △  |
   |プラネット・ウィズ      | BS11  | 12話 | 0   |  -                              | ○  |
   |ヤマノススメ3           | BS11  | 13話 | 0   |  -                              | ○  |
   |ロード オブ ヴァーミリオン| BS11| 12話 | 0   |  -                              | ○  |
   |七星のスバル            | BS11  | 12話 | 1   | 07話で、予告編を誤判定          | △  |
   |天狼 Sirius the Jaeger  | BS11  | 12話 | 0   |  -                              | ○  |
   |殺戮の天使              | BS11  | 12話 | 0   |  -                              | ○  |
   |百錬の覇王と聖約の戦乙女| BS11  | 12話 | 全1 | 全話共通で、予告編を誤判定      | △  |
   |邪神ちゃんドロップキック| BSフジ| 11話 | 0   |  -                              | ○  |

-----------------
## 既知の問題点

+ 番組宣伝の等の CM に 5秒,10秒のものがあるが、それが CM と認識されない。
