---
title: Samba と連携して、Samba経由で再生した動画ファイルの未視聴／視聴済を管理するプログラム
---


## seesaw とは

本プログラムは,
Samba と連携して、Samba経由で再生した動画ファイルの
未視聴／視聴済 を記録し、playList を生成する
ソフトウェアです。

## 背景・目的

PC で録画した動画ファイルを、居間の TV に接続した Fire TV stick や、
タブレットの VLC 等の再生ソフトを使って視聴しているが、
ファイルが溜まってくると、見た／見ていないの管理が煩わしい。
（後でBDに保存するので、見たら消す方式は使えない。）
<br>
そこで、新規のファイルは「未視聴」に、動画再生すれば自動的に「視聴済」
に分類される仕組みを作成した。

<!--more-->

## 動作概要

* 視聴する動画ファイルがあるディレクトリ(*1)を指定する。
* ディレクトリ(*1)検索し、DB に登録する。
  * 新規追加されたファイルは、「未視聴」に分類される。
  * 全てのファイルは、「全て」の含まれる
* 未視聴／視聴済／全て の３つ分類してシンボリックリンク(*2)を生成する。
* samba のアクセスログを監視し、条件を満たすと「未視聴」「視聴済」が入れ替わる。
* 条件とは、
  * 1分以上再生する。(config.rb で変更可) ただし猶予期間中に再度再生開始するとキャンセルされる。
  * 1分以内に、2秒以上の再生を2回繰り返す。


* 視聴する動画があるディレクトリのイメージ (*1)
```
    ├── 番組A
    │     ├── 番組A #01.mp4
    │     ├── 番組A #02.mp4
    │     └── 番組A #03.mp4
    └── 番組B
           ├── 番組B #01.mp4
           └── 番組B #02.mp4
```

* seesaw が生成するシンボリックリンク群のイメージ(*2)
```
    ├── 視聴済
    │     ├── 番組A
    │     │     └── 番組A #01.mp4
    │     └── 番組B
    │            └── 番組B #01.mp4
    ├── 全て
    │     ├── 番組A
    │     │     ├── 番組A #01.mp4
    │     │     ├── 番組A #02.mp4
    │     │     └── 番組A #03.mp4
    │     └── 番組B
    │            ├── 番組B #01.mp4
    │            └── 番組B #02.mp4
    └── 未視聴
           ├── 番組A
           │     ├── 番組A #02.mp4
           │     └── 番組A #03.mp4
           └── 番組B
                  └── 番組B #02.mp4
```

## プログラム構成図

[![]({{site.baseurl}}/img/seesaw_diagram.png){: .ssimg}]({{site.baseurl}}/img/seesaw_diagram.png)


## インストール方法

### seesaw (その１)

* 必要なパッケージを apt install でインストールする。
  * git
  * ruby
  * samba
  * ruby-sqlite3


* インストールするディレクトリに移動し,
   
   ```% git clone https://github.com/kaikoma-soft/seesaw.git```

* 出力先のディレクトリを作成し、次項の config.rb の BaseDir に設定する。

   ```% mkdir -p /data/seesaw ```

* config.sample を $HOME/.config/seesaw/config.rb にコピーして、
  環境に合わせて値を変更する。
  
| 定数名    |  意味                          | 備考       |
|-----------|--------------------------------|------------|
| BaseDir   | seesaw が出力するディレクトリ  |            |
| SambaLog  | samba のアクセスログファイル名 |            |
| TargetDir | 録画ファイルがあるディレクトリ | 複数可     |
| TargetExt | 対象とするファイルの拡張子     | 複数可     |
| Exclude   | 除外するファイル,ディレクトリの正規表現 |複数可,ファイル名可|
| RefreshPeriod | TargetDirの検索周期(秒)    |            |
| DoneTime  |  視聴済にする再生時間(秒)      |            |
| FlipDelay | 反転処理までの遅延時間(秒)     |            |
| LogRotateSize | log ローテーションサイズ(byte)|         |

* 動作確認の為テスト実行する。
  エラーメッセージが出なければ Ctrl-C で中止

   ```
   % sudo touch /var/log/smb_acc.log
   % ruby seesaw.rb
   ```



### samba

* /etc/samba/smb.conf に下記の項を追加する。
  path には、```${BaseDir}/playlist``` を設定する。
  
  ```
  [global]
    wide links = yes
    unix extensions  = no
  [seesaw]
    path = /data/seesaw/playlist
    public = no
    vfs objects = full_audit
    full_audit:prefix = %T|%u
    full_audit:success = open close
    full_audit:failure = none
    full_audit:facility = LOCAL7
    full_audit:priority = NOTICE
  ```
*  変更後にリスタート
  ```% sudo systemctl restart smbd```



### syslog
* /etc/rsyslog.d/50-samba.conf を下記の内容で、新規作成する。

  ```
  local7.*                        -/var/log/smb_acc.log
  ```
*  変更後にリスタート
```% sudo systemctl restart syslog```

* samba 上のファイルにアクセスして、アクセスログに出力される事を確認する。

### seesaw (その2)

* seesaw をバックグラウンドで、実行する。

   ```
   % ruby seesaw.rb -b 
   ```
* OS のリブート時に、起動したい場合は crontab に下記を記述する。

   ```
   @reboot /usr/bin/ruby /home/xxx/yyy/seesaw.rb -b
   ```


## オプションの説明

##### -b, --background

バックグラウンドで実行する。

#####  --done str

str を正規表現として、一致するファイルを視聴済にする。  
例:  ```ruby seesar.rb --done ニュース```

#####  --notyet str
str を正規表現として、一致するファイルを未視聴にする。  
例:  ```ruby seesar.rb --notyet ニュース```

#####    -t
--notyet/done の時に test mode(表示のみ)にする。

##### -k, --kill
バックグラウンドで、動いているseesaw を停止させる。

#####    -l, --loglevel n
log に出力するレベルを指定する。n = 0〜5 でデフォルトは 1

##### -r, --reload
データの再検索を行う。




## 状態遷移の動作説明

samba で共有された動画ファイルに対して、次の操作をした場合に状態遷移が起きる。
<br>
なお操作２は、操作1の終了から FlipDelay で指定した秒数以内で行う操作のこと。

* Long再生 : DoneTime で指定した秒数以上の再生
* Shrot再生 : 2秒以上、DoneTime で指定した秒数未満の再生

| 操作1        | 操作2         | 動作                |
|--------------|---------------|---------------------|
| Long再生     | なし          | 未<->済 の入れ替え  |
| Long再生     | Short再生     | なし(キャンセル)    |
| Long再生     | Long再生      | なし(操作1のキャンセル)    |
| Short再生    | なし          | なし                |
| Short再生    | Short再生     | 未<->済 の入れ替え  |


## 動作確認環境

サーバー側

| マシン       | OS                    |
|--------------|-----------------------|
| PC           |    Ubuntu 18.04.3 LTS |

クライアント側

| マシン       | 再生ソフト                |
|--------------|---------------------------|
| Fire HD 10   | VLC for Android Ver 3.2.6 |
| Fire TV stick| VLC for Android Ver 3.1.4 |


## 連絡先

不具合報告などは、
[GitHub issuse](https://github.com/kaikoma-soft/seesaw/issues)
の方にお願いします。

## リンク

+ [ナマケモノの家 seesaw](http://www.asahi-net.or.jp/~sy8y-siy/src/seesaw.html ){:target="_blank"}
+ [gitHub seesaw](https://github.com/kaikoma-soft/seesaw ){:target="_blank"}
+ [gitHub Pages seesaw](https://kaikoma-soft.github.io/src/seesaw.html ){:target="_blank"}

## ライセンス
このソフトウェアは、Apache License Version 2.0 ライセンスのも
とで公開します。詳しくは LICENSE を見て下さい。



