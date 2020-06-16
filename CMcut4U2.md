---
title: CMcut4U MkⅡ ： Linux(Unix 系 OS) 上で FFmpeg,OpenCV を使って半自動 CMカット
---

## はじめに

本プログラムは、旧版である [CMcut4U](https://github.com/kaikoma-soft/CMcut4U)
を、精度向上等を目的として再構築したものです。

## 旧版からの改良点

* シーンチェンジ検出を導入して、精度の向上
* GUIツールを統合し連携を密にして作業性の向上
* mp4エンコード部をシェルスクリプト化してカスタマイズしやすいように。
* 番組毎に、エンコード方法を選択出来るように。
* 目視チェックの工程も取り込み、作業終了までの一貫性を持たせた。
* 作業終了で、テンポラリファイルを削除、
  TSファイルもゴミ箱に移動させた後、一定時間後に自動削除するように。
* 内部データ構造の整理


## 目的

本プログラムの目的は「PT2等で録画した mpeg2tsファイルを半自動で CMカットし、
希望の形式でエンコードをする」です。


## 特徴

* CM/本編 自動判定の精度は 90% 程度だが、
  番組毎に期待値を設定することにより、
  CMカットが上手く行ったかの見分けを容易としている。

* 自動判定が 上手く行かなかった時は、
  チャプター割り振りの変更を容易にする GUI で修正をする事が出来る

* Linux(Unix 系 OS) 上で動作する。



## プログラムの概要

* TSファイルと、あらかじめ用意したlogo ファイルを入力することにより、
  TSファイルを解析して、CM/本編の自動判定を行う。  
  オプションで、logo 解析を行わず音声のみでカットするモードと、
  CMカットせずに丸ごとエンコードするモードが選択可能。

* 自動判定の精度は 90% 程度なので、番組毎にチャプター数と本編時間の
  期待値を設定し、それと一致するかで一次判定を行う。  
  なお自動判定に失敗するのは、次のような場合
    * 5秒、10秒等の 15秒の倍数でない CM
    * ロゴ付きの CM
    * ロゴなしで次回予告＋提供が 15秒のもの
    * BS の長尺のCM(テレビショッピング)
    * パケットのdrop により、映像と音声にずれがある場合

* 一次判定が NG の場合、専用の GUI を使って CM/本編の割付の修正を行う。

* 一次判定が OK になると、エンコードを行う。

* エンコードの実行部は、自環境に合わせてカスタマイズ出来るように
  シェルスクリプトになっていて、番組毎に選択可能。

* エンコード終了後に、作成した本編、CM ファイルの二次判定を行う。
  二次判定は、動画プレイヤーにより
  本編に CM が混じっていないこと、CM に本編が混じっていないことを目視で
  確認する。
  もし、修正が必要なら、修正GUI に戻って修正する。  
  OK ならば、TSファイル、作業ファイルを削除して、終了とする。
  

<!--more-->

## 実行に必要な環境

* Ubuntu 18.04.1 LTS (多分Unix系ならなんでも)
* ruby  2.5.1 以上
* ruby-gtk2 3.2.4
* ruby-wave
* python 2.7 以上
* Opencv 3.4.1
* ffmpeg(ffprobe) 3.4.6
* mpv 0.27.2
* gimp ver.2.8.22
  (logoファイルの抽出に使用。矩形領域の切り抜きができれば何でも可)
* X window (GUIツール の実行に必要)


## インストール

下記のコマンド例は、Ubuntu でのものです。
他の OS の場合は、コマンドやパッケージ名は適宜変更して下さい。

### ruby
```
% sudo apt install ruby
% sudo apt install ruby-gtk2
% sudo gem install wav-file
```

### python
```
% sudo apt install python-dev python-numpy
```

### ffmpeg & mpv
```
% sudo apt install ffmpeg
% sudo apt install mpv
```

### opencv  
opencv は公式パッケージには無いので、
[opencvの 公式ページ](https://docs.opencv.org/master/d7/d9f/tutorial_linux_install.html)
の手順に従ってソースからインストール。
```
% sudo apt install cmake git libgtk2.0-dev pkg-config libavcodec-dev libavformat-dev libswscale-dev
% sudo apt install libtbb2 libtbb-dev libjpeg-dev libpng-dev libtiff-dev libdc1394-22-dev
% mkdir tmp ; cd tmp
% git clone https://github.com/opencv/opencv.git
% git clone https://github.com/opencv/opencv_contrib.git
% cd opencv ; mkdir build ; cd build
% cmake -D CMAKE_BUILD_TYPE=Release -D CMAKE_INSTALL_PREFIX=/usr/local ..
% make
% sudo make install
```


### 本ソフト

1. インストールするディレクトリを決めて下記を実行する。
   ( 例:~/video/CMcut4U2  )
    ```
    % mkdir -p  $HOME/video/CMcut4U2
    % cd $HOME/video/CMcut4U
    % git clone https://github.com/kaikoma-soft/CMcut4U-Mk2.git .
    ```

1. 環境変数 PATH に上記のディレクトリを追加する。

1. config.rb を $HOME/.config/CMcut4U2/config.rb にコピーし、
   その中身を自分の環境に合わせて書き換える。
   
 | パラメータ名 | 意味                                       |
 |--------------|--------------------------------------------|
 | Top          | TSファイル等のデータを格納するディレクトリ |
 | FFMPEG_BIN   | ffmpeg の実行ファイル名                    |
 | PYTHON_BIN   | python の実行ファイル名                    |
 | NomalSize    | エンコード後の本編画面サイズ               |
 | CMSize       | エンコード後の CM 画面サイズ               |
 | ToMp4        | エンコードスクリプトのデフォルト設定       |
 | MpvOption    | mpv 起動時のオプション文字列               |
 | DelTSZero    | TSファイル削除時に、0byte のファイルを残すか |
 | FadeOut      | チャプターの継ぎ目にFadeOutを挿入するかのデフォルト値 |
 | FadeOutTime  | FadeOutの時間(秒)                                     |
 | Autoremove   | 作業ディレクトリの自動削除を行うか                    |
 | TsExpireDay  | ゴミ箱に移動したTSファイルを何日で削除するか(日)      |

1. 入力ファイル、出力ファイル、作業用ディレクトリを作成する。  
   ( 下記の例は、Top が $HOME/video の場合 )
    ```
    % mkdir $HOME/video
    % cd $HOME/video
    % mkdir TS mp4 logo work
    ```

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



## 使用方法

1. 録画した TS ファイルを「TS/番組名」ディレクトリの下に置く。

1. logoファイルが無ければ、先に作成する。
   ( 後述する [logoファイルの作成方法](#create_logo) を参照して下さい。)

1. 初回は、番組毎のパラメータ設定で、ロゴファイル等の設定を行う。
  ```sh
  % CMcut4U2.rb --paraedit
  ```

1. TSファイルの解析を行う。
  ```sh
  % CMcut4U2.rb
  ```

1. 初回は、期待値が設定されていないので必ず NG になる。  
   そこで修正GUI を起動し、チャプターの CM/本編が正しくなるまで修正を
   行い、最後に「現在の計算値を期待値として設定」ボタンを押し、
   「終了」する。

1. 再度コマンド実行すると、エンコードまで実行される。
  ```sh
  % CMcut4U2.rb
  ```

1. 目視による最終チェックを行う。
  ```sh
  % CMcut4U2.rb --viewchk
  ```
  動画プレイヤー(mpv)で、本編とCM が再生されるので、
  本編に CM が混じっていないこと、CM に本編が混じっていないことを目視で
  チェックする。
  もし、変更が必要なら、修正GUI に戻って修正する。  
  OK ならば、TSファイル、作業ファイルを削除して、終了とする。



## logo ファイルの作成方法 <a name="create_logo"></a>

1. 番組毎のパラメータ設定で、ロゴの位置を指定する。
   1. パラメータ設定ダイアログを起動する。
    ```sh
    % CMcut4U2.rb --paraedit
    ```
    1. 対象ディレクトリを選択する。
    1. ロゴの位置(右上、右下、左下、左上)を選択し、保存してから閉じる。

1. CMcut4U2.rb -\-logo を実行する。 必要なら -\-regex で対象を絞る。
    ```sh
    % CMcut4U2.rb --logo
    ```
    1. スクリーンショット生成の後、ビューアーで画像が表示されるので、
       ロゴマークが明瞭なものを探し、その画像を保存する。  
       ビューアーの操作方法は下記の通りです。

         | キー      | 意味                                       |
         |-----------|--------------------------------------------|
         | j         | 60コマ( 30秒)飛ぶ                          |
         | k         | 60コマ( 30秒)戻る                          |
         | s         | 現在の画像をカレントディレクトリに保存する。|
         | b         | 前のコマ( 0.5秒)に戻る                     |
         | q         | 終了                                       |
         | 上記以外  | 次のコマ( 0.5秒)に進む                     |
    1. 保存した画像の下部（白黒＋強調）部分のロゴマークを
        を 画像処理ソフト(ここでは gimp) を使って、
        最小限の大きさで矩形領域の切り抜き
        (「ツール」-> 「変形ツール」-> 「切り抜き」)をする。
    1. 「画像」-> 「モード」-> 「グレイスケール」で、グレイスケール化する。
    1. logo ディレクトリの下に、名前を付けて画像を保存する。


## 実行コマンドの説明

+ CMcut4U
    - 機能  
      TS ディレクトリの tsファイルに対して解析を行い、
      CMカット、エンコードを行う。
    - オプション

      | short  |   long    |      意味  (詳細はリンク先)        |
      |---|----------------|-----------------------------------------|
      |-C | -\-config file | [configファイルの指定](#config) |
      |-O | -\-odir dir    | [出力Dirの指定](#odir)        |
      |-W | -\-wdir dir    | [作業用Dirの指定](#wdir)
      |-L | -\-logodir dir | [logo Dirの指定](#logodir)
      |-R | -\-regex str   | [処理対象ファイルの指定(正規表現](#regex))
      |-d | -\-debug       | [debug mode](#debug)
      |-F | -\-force       | [自動判定処理の強制実行](#force)
      |-c | -\-co          | [自動判定処理のみ](#co)
      |   | -\-nc          | [キャッシュデータを使わない](#nc)
      |-f | -\-fix         | [チャプター修正ダイアログの起動](#fix)
      |-n | -\-ngfix       | [chk が NG の時、チャプター修正ダイアログを起動](#ngfix)
      |-p | -\-paraedit    | [パラメータ設定ダイアログを起動](#paraedit)
      |-P | -\---emptyPara | [パラメータが未設定のものだけパラメータ設定ダイヤログを起動](#emptyPara)
      |-v | -\-viewchk     | [目視チェックで go/no go を決定](#viewchk)
      |   | -\-tomp4 shell | [mp4 エンコードするスクリプトの指定](#tomp4)
      |-a | -\-autoremove  | [作業ディレクトリの自動削除](#autoremove)
      |-E | -\-forceEnc    | [期待値照合の結果を無視してエンコードを実行](#forceEnc)
      |-l | -\-logo        | [ロゴ作成モード](#logo)

        - -\-config file <a name="config"></a>

          定数定義ファイル(config.rb) のパスを指定する。
          読み込みの優先順位は次の通り。
          1. -\-config で指定したファイル
          1. 環境変数 CMCUT4U2_CONF で指定したファイル
          1. $HOME/.config/CMcut4U2/config.rb

        - -\-odir dir <a name="odir"></a>

          config.rb 中の 定数 Outdir を指定したディレクトリに変更する。

        - -\-wdir dir <a name="wdir"></a>

          config.rb 中の 定数 Workdir を指定したディレクトリに変更する。

        - -\-logodir dir <a name="logodir"></a>

          config.rb 中の 定数 LogoDir を指定したディレクトリに変更する。

        - -\-regex str  <a name="regex"></a>

          処理対象を、str で指定した正規表現に subdir/TSファイル名が
          一致したものだけに限定する。
          正規表現は ruby の文法と同じ。
          
          例   -\-regex タモリ  => ブラタモリ、タモリ倶楽部に一致
           
        - -\-debug <a name="debug"></a>

          デバックモードで実行する。
          
        - -\-force <a name="force"></a>

          通常出力ファイルが既に存在すれば CM自動判定処理は飛ばすが、
          それを強制的に実行させる。

        - -\-co <a name="co"></a>

          CM自動判定処理まで実行するが、その後のエンコード処理は行わない。
          
        - -\-nc <a name="nc"></a>

          workdir のCM自動判定処理の中間データ(キャッシュ)を使わないで
          再実行する。

        - -\-fix <a name="fix"></a>

          チャプター修正ダイアログの起動する。
          対象を絞る場合は、-\-regex を併用する。

          使い方は、
            + 対象のディレクトリ、TSファイルを選択する。
            + 「mpv 起動/終了」ボタンを押すと mpv が起動する。
               mpv が起動した状態で、表中に時間の欄をマウスでクリックすると、
               mpv がその時間まで seek するので、自動判定の結果が正しいか目視で
               確認する。
            + 誤判定が見つかったの行の fix の欄をマウスでクリックし、
              本編、CM  を選択する。
            + 設定し終わったら「変更書き込み」ボタンを押し、
              画面下部の結果表示が○になっている事を
             確認して「終了」する。

            ![修正GUI画面の例]({{site.baseurl}}/img/fixGUI2.png)
          
        - -\-ngfix <a name="ngfix"></a>

         判定結果が NG のファイルにのみチャプター修正ダイアログの起動する。

        - -\-viewchk   <a name="viewchk"> </a>

          目視チェックで go/no go を決定
          
        - -\-paraedit <a name="paraedit"> </a>

          番組毎に設定するパラメータを入力するパラメータ設定ダイアログを
          起動する。  
          対象を絞る場合は、-\-regex を併用する。

          使い方は、
           + 対象のディレクトリを選択する。
           + パラメータを入力する。
           + 「保存」ボタンを押し、値をファイルに書き込む。
           + 「閉じる」ボタンを押し、終了する。

           ![GUI画面の例]({{site.baseurl}}/img/para.png)

        - -\-emptyPara <a name="emptyPara"> </a>

          パラメータが未設定のものだけパラメータ設定ダイヤログを起動する。

        - -\-tomp4 shell <a name="tomp4"></a>

          mp4 エンコードするスクリプトの指定する。 優先順位は  
          引数 > パラメータファイル > config
          
        - -\-autoremove  <a name="autoremove"></a>

           作業ディレクトリの自動削除の実行／非実行を反転させる。
           デフォルトの値は config.rb 中の Autoremove の値

        - -\-forceEnc    <a name="forceEnc"></a>

          期待値照合の結果を無視してエンコードを実行する。

        - -\-logo  <a name="logo"></a>

          ロゴ作成モードで実行する。
          対象を絞る場合は、-\-regex を併用する。



           

## カスタマイズ

#### ffmpeg エンコード

ディレクトリ libexe 以下に、エンコードに使うshellスクリプトがある。
ffmpeg のオプション等を変えたい場合は、それらを直接書き換えるか、
既存の物をコピーして変更を加える。


## 連絡先

不具合報告などは、
[GitHub issuse](https://github.com/kaikoma-soft/CMcut4U-Mk2/issues)
の方にお願いします。

## リンク

+ [mpv ドキュメント](https://mpv.io/manual/master/)
+ [ナマケモノの家 CMcut4UMk2](http://www.asahi-net.or.jp/~sy8y-siy/CMcut4UMk2/ ){:target="_blank"}
+ [gitHub CMcut4UMk2](https://github.com/kaikoma-soft/Mcut4U-Mk2 ){:target="_blank"}
+ [gitHub Pages CMcut4U2](https://kaikoma-soft.github.io/CMcut4U2.html ){:target="_blank"}

