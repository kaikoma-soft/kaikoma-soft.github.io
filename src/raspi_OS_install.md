title: Raspberry Pi 3B+ ＋ PX-Q3U4 で録画サーバー (OSインストール)



## OS (Raspbian) インストール


1. 作業PC上で公式ページ <https://www.raspberrypi.org/downloads/raspbian/> から
Raspbian Stretch Lite をダウンロード

1. 上記の img を dd で SDカードに書き込む
```
$ dd if=2018-11-13-raspbian-stretch-lite.img of=/dev/sdd bs=4M
```
  
1. 上記の SD の boot領域をマウントして ssh の設定
```  
$ mount /dev/sdd1 /mnt/sdd
$ touch /mnt/sdd/ssh
```  

1. boot領域をアンマウント。
```  
$ umount /dev/sdd1
```  
1. SDカードを抜き、raspberry Pi に挿して、電源投入。

1. しばらく待って作業用マシンから ssh でログイン。
    mDNSが動いているので
    ```
    $ ssh pi@raspberrypi.local
    ```
    で可能。初期パスワードは、raspberry
   
1. raspi-config を実行して、passwd, locale,time zone の設定。

1. SD カードの root領域のサイズを確認
 
```
 $ fdisk -l /dev/mmcblk0

Device         Boot Start      End  Sectors  Size Id Type
/dev/mmcblk0p1       8192    98045    89854 43.9M  c W95 FAT32 (LBA)
/dev/mmcblk0p2      98304 31116287 31017984 14.8G 83 Linux
```

1. 外付けUSB HDD の追加して、そちらをメインに。(SDカードはbootだけ)
   USB HDD は /dev/sda で認識.
  
```
  $ fdisk /dev/sda  
```
で、下記のようにパーティションの設定 ( 全部で 1T )  
```
   /dev/sda1  16G           /
   /dev/sda2  残り(968G)    /data
```

```
  $ mkfs.ext4 /dev/sda1
  $ mkfs.ext4 /dev/sda2
```
  
1. /dev/sda1 を /mnt/root にマウントして、
  dump & restore で、SDの root 領域を HDD にコピー。

```
  $ mkdir /mnt/root
  $ mkdir /mnt/root/data
  $ mount /dev/sda1 /mnt/root 
  $ apt install dump
  $ dump -0f - /    | ( cd /mnt/root ; restore -rf - )
```

1. PARTUUID を調べる。(下記のどちらか)

```
$ ls -l /dev/disk/by-partuuid
lrwxrwxrwx 1 root root 10 12月 21 10:32 9049d880-01 -> ../../sda1
lrwxrwxrwx 1 root root 10 12月 21 10:32 9049d880-02 -> ../../sda2
lrwxrwxrwx 1 root root 15 12月 21 10:32 be23acea-01 -> ../../mmcblk0p1
lrwxrwxrwx 1 root root 15 12月 21 10:32 be23acea-02 -> ../../mmcblk0p2

$ blkid 
/dev/mmcblk0p1: LABEL="boot" UUID="9304-D9FD" TYPE="vfat" PARTUUID="be23acea-01"
/dev/mmcblk0p2: LABEL="rootfs" UUID="29075e46-f0d4-44e2-a9e7-55ac02d6e6cc" TYPE="ext4" PARTUUID="be23acea-02"
/dev/sda1: LABEL="root" UUID="f4f8761e-5899-468f-82c5-c32922928adf" TYPE="ext4" PARTUUID="9049d880-01"
/dev/sda2: LABEL="data" UUID="ff86fdae-92e5-49c5-b968-50da38116c34" TYPE="ext4" PARTUUID="9049d880-02"
```

1. /mnt/root/etc/fstab を修正。
   SDカードの be23acea-02 から HDDの 9049d880-01 に書き換える
```
proc            /proc           proc    defaults          0       0
PARTUUID=be23acea-01  /boot           vfat    defaults          0       2
PARTUUID=be23acea-02  /               ext4    defaults,noatime  0       1
```
↓

```
proc            /proc           proc    defaults          0       0
PARTUUID=be23acea-01  /boot           vfat    defaults          0       2
PARTUUID=9049d880-01  /               ext4    defaults,noatime  0       1
PARTUUID=9049d880-02  /data           ext4    defaults,noatime  0       1
```


1. /boot/cmdline.txt のPARTUUIDも同様に書き換える。

```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=9049d880-01 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait
```

1. swap の大きさと場所を変える。 /etc/dphys-swapfile
```
 CONF_SWAPFILE=/data/swap
 CONF_SWAPSIZE=1024
```

#### メモ

* 最初は SDカード無し USB HDD のみの構成で構築したが、
   その状態で PX-Q3U4 を接続すると boot しない為、
   仕方なく SDカードから bootして root領域は HDD に置くようにした。
   
* SDカードの残りエリアがもったいないので、HDD の root 領域のバックアップ
  として活用する。
  その為に、HDD の root領域を SDカードのサイズに合わせた。

  バックアップ用のスクリプトの例
```
mkfs.ext4 /dev/mmcblk0p2
mount /dev/mmcblk0p2 /mnt/root
dump -0f - /  | ( cd /mnt/root ; restore -rf - )
umount  /mnt/root
```


-----------------
## OS のダイエット

1. ipv6 の停止
    1. /boot/cmdline.txt に "ipv6.disable=1" を追記
```
dwc_otg.lpm_enable=0 console=serial0,115200 console=tty1 root=PARTUUID=9049d880-
01 rootfstype=ext4 elevator=deadline fsck.repair=yes rootwait ipv6.disable=1
```
    1. kernel モジュールの削除(後述)

1. 固定IP の設定
    1.  /etc/network/interfaces に下記を追記
```
allow-hotplug eth0
iface eth0 inet static
    address 192.168.1.XX
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.YY
```

   最近は /etc/dhcpcd.conf の方で設定するようだか、dhcpcdサービスを止めたら
   意味が無いので、こちらで設定する。

1. 不要なサービスの停止
```
$ systemctl disable avahi-daemon.service
$ systemctl disable bluetooth.service 
$ systemctl disable cups.service 
$ systemctl disable dhcpcd.service 
$ systemctl disable hciuart.service 
$ systemctl disable rsync.service 
$ systemctl disable syslog.socket 
$ systemctl disable rsyslog.service 
$ systemctl disable triggerhappy.socket 
$ systemctl disable triggerhappy.service
$ systemctl disable wifi-country.service
```
                 
1. 不要なカーネルモジュールの削除
   /etc/modprobe.d/raspi-blacklist.conf に下記を追記(無ければ作る)
```
blacklist brcmfmac       # Wifi
blacklist snd_bcm2835    # サウンド
blacklist joydev         # joy stick
blacklist ip_tables      # パケットフィルタリング
blacklist ipv6
```

-----------------
## px4 ドライバーのインストール

1. 必要なパッケージをインストール
```
  $ apt install raspberrypi-kernel-headers dkms git
```

1. [配布元](https://github.com/nns779/px4_drv)
からソースをダウンロード
```
  $ git clone https://github.com/nns779/px4_drv.git
```

1. ドキュメントに従ってインストール。

1. ドキュメントにはないが、raspbian の場合は、
   1. /etc/modules に px4_drv を追記
   
   1. /boot/cmdline.txt に coherent_pool=4M
      ( これが無いと同時に 4TS が出来ずに 2TS になる。)

1. reboot して lsmod | grep px4 してドラーバーはload されている事を確認。
  PX-Q3U4 を接続して、/dev 以下にデバイスが現れるか確認。
  
```
$ ls -l /dev/px*
crw-rw-r-- 1 root video 244, 0 12月 21 11:31 /dev/px4video0
crw-rw-r-- 1 root video 244, 1 12月 21 11:31 /dev/px4video1
crw-rw-r-- 1 root video 244, 2 12月 21 11:31 /dev/px4video2
crw-rw-r-- 1 root video 244, 3 12月 21 11:31 /dev/px4video3
crw-rw-r-- 1 root video 244, 4 12月 21 11:31 /dev/px4video4
crw-rw-r-- 1 root video 244, 5 12月 21 11:31 /dev/px4video5
crw-rw-r-- 1 root video 244, 6 12月 21 11:31 /dev/px4video6
crw-rw-r-- 1 root video 244, 7 12月 21 11:31 /dev/px4video7
```


