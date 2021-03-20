---
title: raspirec パケットチェック機能
---


## はじめに

ここでは、raspirec のオプション機能である「パケットチェック機能」
について説明する。

## パケットチェック機能とは

パケットチェック機能とは,録画した TS ファイルの健全性(drop や error) を
[tspackectchk]( https://github.com/kaikoma-soft/tspacketchk ){:target="_blank"}
を使ってチェックするものです。
<br>
結果は、「録画済み一覧」の画面に表示されます。


[![]({{site.baseurl}}/img/packetchk2.png){: .ssimg}]({{site.baseurl}}/img/packetchk2.png)


## 機能を有効にするには

* tspacketchk をインストールする。
<br>
インストール方法は,
[https://github.com/kaikoma-soft/tspacketchk]( https://github.com/kaikoma-soft/tspacketchk )
を参照。

*  config.rb 中に下記の定数を設定する。
   * PacketChk_enable を true 
   * PacketChk_cmd に tspacketchk のインストール先を設定
   * PacketChk_rate は、実測値からフィードバックして設定
   * PacketChk_threshold には、画面上で赤枠を付ける閾値を設定
   * PacketChk_opt は任意

* config.rb の記述例
```
PacketChk_enable     = true                         # true = 有効にする
PacketChk_cmd        = "/usr/local/bin/tspacketchk" # path
PacketChk_opt        = "-s 1 "                      # オプション
PacketChk_threshold  = 2                            # エラーと判定する閾値
PacketChk_rate       = 100                          # 想定速度 ( Mbyte/秒 )
```

 
* raspirec を再起動する。



##  Memo
  * チェック動作は、リアルタイムではなく空き時間に行うので、
    結果が出るまでに時間差がある。
  * config.rb の変更を反映させるには、再起動が必要。
  * 結果の数字をクリックすると、検査結果が表示される。
