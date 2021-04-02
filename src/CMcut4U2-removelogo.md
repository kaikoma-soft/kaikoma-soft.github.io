---
title: CMcut4U2 のロゴ消し機能
---


## はじめに

CMcut4U2 Ver 0.3.0 で追加されたロゴ消し機能についての説明です。


## 使い方の概略

1. config.rb に Ver 0.3.0 で[追加された定数](#config)を追記する。
1. あらかじめ、放送局毎にロゴのマスクデータを作成しておく。[作り方は後述](#logomask)
1. 番組毎にパラメータ設定ダイアログで、上記のロゴのマスクデータを指定する。
   また、必要ならばぼかしフィルターを選択する。

以上で、エンコード時に ffmpeg の removelogo + ぼかしフィルター処理を実行する。
<br>

 * ロゴのマスクデータを指定した時のみ、ぼかしフィルターは有効になる。
 * フィルタチェインは下記の様なものになる。
<pre>
-vf split[a][b],[a]removelogo=ABCD-194x101+1220+30.png[a1],[a1]crop=194:101:1220:30[a2],[a2]unsharp=5:5:-1.0:5:5:-1.0[a3],[b][a3]overlay=1220:30
</pre>


## config.rb に追加された定数 <a name="config"></a>

<pre>
RmLogoDir      = Top + "/rmlogo"
RmLogoBlurList = {
  :null      => "null",                        # 素通し
  :smartblur1 => "smartblur=1:1:0:1:1:0",
  :smartblur2 => "smartblur=2:1:0:2:1:0",
  :unsharp1   => "unsharp=5:5:-1.0:5:5:-1.0",
  :unsharp2   => "unsharp=7:7:-1.5:7:7:-1.5",
  }
RmLogoBlurDefault = :unsharp1
</pre>

| 定数名           |   意味            |
|------------------|-------------------|
| RmLogoDir        | ロゴのマスクデータを格納するディレクトリ |
| RmLogoBlurList   | ぼかしフィルターのリスト。|
| RmLogoBlurDefault | RmLogoBlurList 中のからパラメータ設定ダイアログの初期値とするものを指定 |

* リストの中身はカスタマイズ自由。
* フィルターやパラメータの意味は ffmpeg のドキュメントを参照。
  * [smartblur](https://ffmpeg.org/ffmpeg-filters.html#smartblur-1)
  * [unsharp](https://ffmpeg.org/ffmpeg-filters.html#unsharp-1)


## ロゴマスク画像ファイルの作成方法 <a name="logomask"></a>

黒地にロゴしか入っていないロゴマスク画像ファイルを得るため、下記の作業を行う。


1. ロゴを採取する TSファイルを用意する。
1. 上記の TSファイルの解像度を ffprobe で調べる。(下記の赤字 or 太字の部分)
<pre>
    % ffprobe TSファイル名
    ....
        Stream #0:1[0x111]: Video: mpeg2video (Main) ([2][0][0][0] / 0x0002), yu v420p(tv, bt709, top first), <strong>1440x1080</strong> [SAR 4:3 DAR 16:9], 29.97 fps, 29.97 tbr, 90k tbn, 59.94 tbc
        Side data:
          cpb: bitrate max/min/avg: 20000000/0/0 buffer size: 9781248 vbv_delay: N/A
        Stream #0:2[0x112]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 48000 Hz, stereo, fltp, 189 kb/s
        Stream #0:3[0x114]: Subtitle: arib_caption (Profile A) ([6][0][0][0] / 0x0006)
        Stream #0:4[0x115]: Data: bin_data ([6][0][0][0] / 0x0006)
        Stream #0:5[0x840]: Unknown: none ([13][0][0][0] / 0x000D)
    ....
</pre>



1. TS ファイルを mpv 等のプレイヤーで再生し、
   ロゴが鮮明に見えるタイミングで、スクリーンショットをとる。
1. スクリーンショットの画像を gimp で編集する。
   1. 解像度が TS ファイルのものと異なる場合は、TSファイルの解像度に合わせる。
      <br>
      「画像」→「画像の拡大・縮小」
   1. グレイスケールに変換する。
      <br>
      「画像」→「モード」→「グレイスケール」
   1. ロゴの部分以外を黒(#000000) で塗りつぶす。
       <br>
      1. ロゴの部分を矩形領域で選択。
      1. 「選択」→「選択範囲の反転」
      1. 「編集」→「描画色で塗りつぶす」
   1. 適当なフィルタや描画ツールを使って、ロゴの周辺をきれいに整形する。
   1. ぼかしフィルターの為にロゴの位置情報を調べる。
      <br>
      ロゴを矩形選択で囲む。→
      ツールオプションに座標データが表示されているのでそれをメモしておく。
      必要なものは、左上の座標と、高さ×幅で、
      この部分にぼかしフィルターが掛けられる。      
   1. png ファイルを出力する。
      <br>
      「ファイル」→「名前を付けてエクスポート」→
      ファイル名を指定してエクスポート
      <br>
      ファイル名は、次の書式にする。

      | 画像フォーマット | png グレイスケールを推奨 (他の形式は動作未確認) |
      | 書式   | ABCDEFG-WxH+X+Y.png |
      | 例     |  TBS-113x30+1868+66.png |
      | ABCDEFG| 任意の文字列(ただし ```:```は避ける)
      | W , H  | ロゴの幅、高さ (整数)  |
      | X , Y  | ロゴの左上の位置(整数) |
      | 置き場所 | RmLogoDir で指定したディレクトリ |

    1. 確認
   <br>
     作成したマスク画像ファイルを確認するには下記のコマンドを実行する。
```
     ffplay -i TSファイル -vf removelogo=マスク画像ファイル名
```

### 注意点
  1. ロゴの位置情報は、使用するぼかしフィルターによるが、ロゴより一回り大きくとると良い?
  1. 放送局によっては、ロゴに陰影を付けているものがある。
     その場合は影の部分もロゴの領域としなければならない。
      



## 参考 Link
* [FFMPEG できれいにロゴを消す方法](https://nico-lab.net/removelogo_for_high_quality_with_ffmpeg/)
* [FFmpeg Documentation](https://ffmpeg.org/documentation.html)