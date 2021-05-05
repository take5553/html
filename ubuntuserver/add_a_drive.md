# ドライブ増設

Ubuntu Serverはファイルサーバーとして使うつもりなのでそのためのドライブを拡張する。[前回](lvm.html)やったLVMシステムは使わずに、独立して増設する。LVMはあくまでも「Ubuntuインストール時にほいほい進んでたら勝手にそうなっちゃった」から対処しただけ。

今回使うのは1TBのSSD。HDDに比べて可動部が無いから壊れにくいと思ったのと、自分のファイルストレージはそんなに容量を必要としないから。何年も溜め込んでいるメインPCのDドライブは500GBぐらいで、そんなにガバガバ増えていかないいでしょ。

参考：[HDD増設手順メモ - Qiita](https://qiita.com/bwtakacy/items/c181f661e8655c42d85a)

## 認識確認

USB3.0かSATAか何かで物理的にサーバーPCに接続してから、ちゃんとOSに認識されたかどうかを確認。

~~~shell
$ sudo fdisk -l
Disk /dev/loop0: 55.46 MiB, 58142720 bytes, 113560 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop1: 69.9 MiB, 73277440 bytes, 143120 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop2: 55.39 MiB, 58073088 bytes, 113424 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop3: 31.9 MiB, 32600064 bytes, 63672 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop4: 70.39 MiB, 73797632 bytes, 144136 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/loop5: 32.28 MiB, 33841152 bytes, 66096 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/mmcblk0: 57.63 GiB, 61865984000 bytes, 120832000 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: gpt
Disk identifier: 0C4F1CC4-25EE-4910-A740-C134B1408B10

Device           Start       End   Sectors  Size Type
/dev/mmcblk0p1    2048   1050623   1048576  512M EFI System
/dev/mmcblk0p2 1050624   3147775   2097152    1G Linux filesystem
/dev/mmcblk0p3 3147776 120829951 117682176 56.1G Linux filesystem


Disk /dev/mapper/ubuntu--vg-ubuntu--lv: 56.12 GiB, 60251176960 bytes, 117678080 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes


Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: 500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
~~~

最後の`Disk`がどうもそれっぽい。

## パーティション作成

まっさらなドライブは本当に何も無いので、最初にパーティションを作成する。

~~~shell
$ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.34).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.

Device does not contain a recognized partition table.
Created a new DOS disklabel with disk identifier 0xe09162dc.

Command (m for help):
~~~

対話形式で進んでいく。

とりあえずどんなコマンドがあるのか確認。

~~~
Command (m for help): m

Help:

DOS (MBR)
a   toggle a bootable flag
b   edit nested BSD disklabel
c   toggle the dos compatibility flag

Generic
d   delete a partition
F   list free unpartitioned space
l   list known partition types
n   add a new partition
p   print the partition table
t   change a partition type
v   verify the partition table
i   print information about a partition

Misc
m   print this menu
u   change display/entry units
x   extra functionality (experts only)

Script
I   load disk layout from sfdisk script file
O   dump disk layout to sfdisk script file

Save & Exit
w   write table to disk and exit
q   quit without saving changes

Create a new label
g   create a new empty GPT partition table
G   create a new empty SGI (IRIX) partition table
o   create a new empty DOS partition table
s   create a new empty Sun partition table
~~~

ここでは`n`でいいらしい。`p`か`e`かを聞かれるけど、何も無いので`p`。特に何も入力せずにEnterを押すとデフォルト値が採用されるらしい。

~~~
Command (m for help): n
Partition type
p   primary (0 primary, 0 extended, 4 free)
e   extended (container for logical partitions)
Select (default p): 
~~~

パーティション番号はデフォルトの`1`。

~~~
Partition number (1-4, default 1): 
~~~

最初と最後のセクター。よく分からんからそのまま。

~~~
First sector (2048-1953525167, default 2048): 
Last sector, +/-sectors or +/-size{K,M,G,T,P} (2048-1953525167, default 1953525167): 
~~~

これでパーティションが作成されたらしい。

~~~
Created a new partition 1 of type 'Linux' and of size 931.5 GiB.
~~~

そのことを確認。コマンドは`p`。

~~~
Command (m for help): p
Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: 500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xea8e3efc

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1        2048 1953525167 1953523120 931.5G 83 Linux
~~~

この`Id`が`83`だったらLinux用。`8e`だったらLinux LVM用らしい。

最後に設定した内容で実行。コマンドは`w`。

~~~
Command (m for help): w
The partition table has been altered.
Calling ioctl() to re-read partition table.
Syncing disks.
~~~

実際に実行されたことを確認。

~~~shell
$ sudo fdisk -l

(略)

Disk /dev/sda: 931.53 GiB, 1000204886016 bytes, 1953525168 sectors
Disk model: 500SSD1
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xea8e3efc

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1        2048 1953525167 1953523120 931.5G 83 Linux
~~~

`Disklabel`、`Disk identifier`、最後の2行が追加された。

## ファイルシステムの作成

とりあえず`ext4`がスタンダード。Garuda Linuxは`btrfs`を採用しているのが特徴。

~~~shell
$ sudo mkfs -t ext4 /dev/sda1
mke2fs 1.45.5 (07-Jan-2020)
Creating filesystem with 244190390 4k blocks and 61054976 inodes
Filesystem UUID: 7c399b2a-e023-4f8e-8596-8d81fb09b141
Superblock backups stored on blocks:
32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
102400000, 214990848

Allocating group tables: done
Writing inode tables: done
Creating journal (262144 blocks): done
Writing superblocks and filesystem accounting information: done
~~~

ファイルシステムがあるから`/home/takeshi/hogehoge.txt`を開け、みたいな指示ができる。ファイルシステムが無いと、セクターどこどこからセクターどこどこまでを開けみたいな指示をしないといけないらしい。

## マウント

普通はUSBメモリなんかでも挿せばすぐ認識するとはならず、マウントということをしてあげないといけない。Windowsなんかは挿せば勝手にマウントする。Windowsで「取り出し」を実行すると、USBメモリが挿さっているのに認識していないという状態になるけど、あれはマウント状態が解除されたもの。

現状ではまだSSD側の準備ができただけで、Linuxが認識をしていない。今後は挿す度にマウントしないと使えない。

先にマウント先のディレクトリを作成する。

~~~shell
$ sudo mkdir /mnt/ssd1
~~~

そこに今準備したSSDをマウントする。

~~~shell
$ sudo mount /dev/sda1 /mnt/ssd1
~~~

これで使えるようになった。

## 起動時自動マウント

サーバーなので基本的には動かしっぱなしだけど、再起動のために起動時に自動でマウントするようにする。

まず今追加したSSDのUUIDというものを調べておく。これはドライブごとにユニークだそうだ。

~~~shell
$ sudo blkid
/dev/mmcblk0p1: UUID="C441-0CBA" TYPE="vfat" PARTUUID="2c127b24-a13b-4986-825a-d56fb0b0de66"
/dev/mmcblk0p2: UUID="27aaf490-266c-41ca-98cd-0fc5d564bc0f" TYPE="ext4" PARTUUID="a409702f-908a-4034-a3f8-15d443df0696"
/dev/mmcblk0p3: UUID="dETlFM-RLGm-LpUJ-1hoG-QnyN-hKBc-QHrt6O" TYPE="LVM2_member" PARTUUID="bd4cafda-f567-4629-9d37-37e11846e108"
/dev/mapper/ubuntu--vg-ubuntu--lv: UUID="04b573bd-e275-4517-9bc6-72c34455bdc2" TYPE="ext4"
/dev/loop0: TYPE="squashfs"
/dev/loop1: TYPE="squashfs"
/dev/loop2: TYPE="squashfs"
/dev/loop3: TYPE="squashfs"
/dev/loop4: TYPE="squashfs"
/dev/loop5: TYPE="squashfs"
/dev/sda1: UUID="7c399b2a-e023-4f8e-8596-8d81fb09b141" TYPE="ext4" PARTUUID="ea8e3efc-01"
~~~

最後の1行が今追加したSSD。なのでこのUUIDをコピーしておく。

設定ファイルに自動マウント情報を記述。

~~~shell
$ sudo nano /etc/fstab
~~~

末尾に以下を記入。

~~~
UUID=7c399b2a-e023-4f8e-8596-8d81fb09b141 /mnt/ssd1 ext4 defaults 0 0
~~~

こういう風に書いても可だけど、UUIDの方がより信頼できるみたい。

~~~
/dev/sda1 /mnt/ssd1 ext4 defaults 0 0
~~~

とにかく、これで再起動したあともちゃんとマウントされる。

参考：[Linux ハードディスクをマウント（mount）する](https://kazmax.zpp.jp/linux_beginner/mount_hdd.html)