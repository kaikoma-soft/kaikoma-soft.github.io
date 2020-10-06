
## 目的

raspirec の動作テストをする為の docker コンテナです。

## 条件

* ホストマシンに、docker がインストールされていること
  <pre>
    % apt install docker.io
  </pre>
* ホスト側に、チューナーカードに対応したドライバーがインストールされていること
  <br>
  ( recpt1/recsvb の切り替えはデバイスファイルを見て自動対応します。 )
  <table>
    <tr>
      <th> チューナーカード名</th>
      <th> ドライバー  </th>
      <th> デバイスファイル </th>
      <th> 録画コマンド </th>
    </tr>
    <tr>
      <td> PT1, PT2 </td>
      <td> pt1_drv </td>
      <td> /dev/pt1video0 </td>
      <td rowspan="3"> recpt1 </td>
    </tr>
    <tr>
      <td> PT3 </td>
      <td> pt3_drv </td>
      <td> /dev/pt3video0 </td>
    </tr>
    <tr>
      <td> Plex社製 PX-Q3U4 等</td>
      <td> px4_drv </td>
      <td> /dev/px4video0 </td>
    </tr>
    <tr>
      <td> PT1, PT2, PT3 </td>
      <td> DVBドライバ(OS標準) </td>
      <td> /dev/dvb/adapter0/frontend0 </td>
      <td> recdvb *1</td>
    </tr>
  </table>
  *1 : recdvb は本家のものではなく recpt1互換の [ dogeel版 recdvb](https://github.com/dogeel/recdvb) を使用します。

* デバイスファイルに適切なアクセス権が付与されていること
   ```
   % ls -l /dev/p*video*
   crw-rw-rw- 1 root video 242, 0 10月  1 20:01 /dev/pt1video0
   crw-rw-rw- 1 root video 242, 1 10月  1 20:01 /dev/pt1video1
   crw-rw-rw- 1 root video 242, 2 10月  1 20:01 /dev/pt1video2
   crw-rw-rw- 1 root video 242, 3 10月  1 20:01 /dev/pt1video3
   ```
* ホスト側の pcscd は停止して置くこと。(Docker内でpcscd を起動します。)


## インストール手順

* 作業用ディレクトリを作成し、Dockerfile 一式をダウンロード
  <pre>
  % mkdir tmp
  % cd tmp
  % git clone https://github.com/kaikoma-soft/docker-raspirec.git .
  </pre>
* setting.sh にパラメータが設定されているので、好みで変更する。
* Docker Image をビルドする
  <pre>
  % sh ./build.sh
  </pre>

* ホスト側の $HOME/docker-raspirec (Docker側の /raspirec )に
  raspirec が参照する config.rb が作成されるので、
  それをエディタで適切な値に書き換える。
  <br>
  最低限必須なのは次のもの。詳細は
  [doc/config.md](https://github.com/kaikoma-soft/raspirec/blob/master/doc/config.md) を参照
   ```
   GR_EPG_channel    : 地デジ EPG 受信局を設定する。
   GR_tuner_num      : 地デジチュナー数
   BSCS_tuner_num    : BSCSチュナー数
   ```

## 起動
   ```
   % sh ./run.sh
   ```

## WEBアクセス
   WEB ブラウザで、http://Dockerホスト名:45678/ へアクセスする。

## 停止
   ```
   % sh ./stop.sh
   ```

## メモ
* recpt1,recdvb 等は /usr/local/bin にインストールされます。
* raspirec は /usr/local/raspirec にインストールされます。
* 録画したTSファイル、DB、log 等は、
  ホスト側の $HOME/docker-raspirec (Docker側の /raspirec )に作られます。
* 通常 docker のイメージが保存される場所は /var/lib/docker 以下ですが、
  容量が足りない場合は、下記の手順で保存場所を変更出来ます。
  * /etc/docker/daemon.json を下記の内容で作成し、
    <pre>
    {
       "data-root": "/ssd1/docker"
    }
    </pre>
   * docker を再起動する。
     <pre>
     % sudo service docker restart
     </pre>
* ドライバーの無効化は、/etc/modprobe.d/blacklist.conf で行う。
  <pre>
  blacklist earth_pt1
  #blacklist pt1_drv
  </pre>

## 動作確認環境

|  ホストOS  |  Ubuntu 18.04.5 LTS |
| チューナー | PT2, PX-Q3U4        |
| ドライバー | pt1_drv,px4_drv,DVBドライバ(OS標準) |
| Docker     | 19.03.6             |
| Docker ベース・イメージ | Ubuntu:20.10 |

## 注意事項

* 動作テスト用なので、ファイルサイズ、セキュリティの考慮はしていません。
  <BR>
  HLSモニタ機能有効にした場合 615MB, しない場合 178MB 程度を使用します。

## リンク
[recdvb](https://github.com/dogeel/recdvb)

## ライセンス
このソフトウェアは、Apache License Version 2.0 ライセンスのも
とで公開します。詳しくは LICENSE を見て下さい。



