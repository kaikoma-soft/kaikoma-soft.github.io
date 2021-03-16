---
title: CMcut4U2 の字幕処理について
---


## はじめに

CMcut4U2 Ver 0.2.0 で追加された字幕処理についての説明です。

## config.rb

字幕処理を有効にするには、config.rb に下記の定義を行う。

```
Subtitling  = true    # 字幕の処理を行う
```

これにより arib字幕が入った TSファイルの場合に下記の処理を行う。
1. TSファイルから字幕データの抽出
1. CMカット情報から、字幕のタイミングを変換
1. エンコード後の mp4 ファイルに字幕データを重畳する。
1. 出力ファイルのフォーマットは mkv になる。



## 制限事項

* 字幕処理を行うには、libaribb24, libass を有効にした
  ffmpeg 4.2 以降が必要です。(後述)
* libaribb24 では
  * arib外字は 〓 になります。
  * 字幕の位置情報は失われます。
  * 位置情報が無いため正しい位置にルビを出力する事が出来ないので、
    ルビを出力していません。
* 字幕のタイミングがズレる事があるので、プレイヤー側で調整するか、
  パラメータ設定ダイアログの「字幕のタイミング調整」で調整する必要があります。


## arib字幕処理が可能な ffmpeg の作り方

* 対象は ffmpeg release 4.3 
* CMcut4U2 に必要な機能は、下記のものとする。(その他は必要に応じて適宜)
  * x254
  * x265
  * nvenc
  * arib 字幕
  * mpeg2video
  * vaapi
  * vdpau
  * cuda
* インストールするディレクトリは /usr/local/ffmpeg/4.3
* OS は ubuntu 20.04 LTS を想定

### 必要なパッケージのインストール
```
% sudo apt install make autoconf libtool libaribb24-dev nasm pkg-config libx265-dev libx264-dev vainfo libvdpau-dev libva-dev nvidia-cuda-dev libass-dev
```



### NVidia headers のインストール

```
% git clone https://github.com/FFmpeg/nv-codec-headers.git
% cd nv-codec-headers
% sudo make install PREFIX=/usr
```


### ffmpeg のインストール

```
% git clone https://github.com/FFmpeg/FFmpeg.git --depth 1 -b n4.3
% cd FFmpeg
% ./configure --prefix=/usr/local/ffmpeg/4.3 --enable-gpl --enable-libx265 --enable-libx264  --enable-libaribb24 --enable-libass --enable-version3
% make -j 4
% sudo make install
```

### 必要な機能が有効になっているかの確認
```
% ./ffmpeg -hide_banner -encoders 2>&1 | egrep "(libx264|libx265|nvenc_h264|aac)"
 V..... libx264              libx264 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 (codec h264)
 V..... libx264rgb           libx264 H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10 RGB (codec h264)
 V..... nvenc_h264           NVIDIA NVENC H.264 encoder (codec h264)
 V..... libx265              libx265 H.265 / HEVC (codec hevc)
 A..... aac                  AAC (Advanced Audio Coding)

% ./ffmpeg -hide_banner -decoders 2>&1 | egrep "(arib|ASS|srt)"
 S..... libaribb24           libaribb24 ARIB STD-B24 caption decoder (codec arib_caption)
 S..... ssa                  ASS (Advanced SubStation Alpha) subtitle (codec ass)
 S..... ass                  ASS (Advanced SubStation Alpha) subtitle
 S..... srt                  SubRip subtitle (codec subrip)

% ./ffmpeg -hide_banner -hwaccels
Hardware acceleration methods:
vdpau
cuda
vaapi
```

### config.rb に下記の定義を行う。

```
FFMPEG_BIN  = "/usr/local/ffmpeg/4.3/bin/ffmpeg"
```



## 参考 Link
* [FFMPEG に NVENC（CUDA）をインストールする](https://nico-lab.net/installing_cuda_with_ffmpeg/)
* [ARIB字幕をDEMUXする LIBARIBB24](https://nico-lab.net/libaribb24_with_ffmpeg/)
* [libaribb24](https://github.com/nkoriyama/aribb24)
* [ARIB外字 - Wikipedia](https://ja.wikipedia.org/wiki/ARIB外字)