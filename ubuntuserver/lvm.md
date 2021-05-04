# 64GBのストレージが何故か32GBしか認識されない（LVM解説）

インストール時にLVMがどーのこーの言われて、32GBだぞみたいなことを言われた。いやいや、何を言ってんねん、これ64GBのストレージあるぞ。

とりあえずSSH接続はできているものとする。

## 現状把握

~~~shell
$ sudo pvs
  PV             VG        Fmt  Attr PSize  PFree
  /dev/mmcblk0p3 ubuntu-vg lvm2 a--  56.11g 28.05g
$ sudo vgs
  VG        #PV #LV #SN Attr   VSize  VFree
  ubuntu-vg   1   1   0 wz--n- 56.11g 28.05g
$ sudo lvs
  LV        VG        Attr       LSize   Pool Origin Data%  Meta%  Move Log Cpy%Sync Convert
  ubuntu-lv ubuntu-vg -wi-ao---- <28.06g
~~~

この`PFree`、`VFree`の値の分だけ余っているということらしい。普通にUbuntu Serverをインストールしただけなのになぜそんなことになるのか。全部使い切れよ。

## LVMとは

Logical Volume Management。詳しいことはよく分からんけど、物理ドライブの容量にとらわれない論理ドライブ的なものを作っちまおうぜ、というもの。

正確ではないけど、自分はこう認識した。

* PV(Phisical Volume)・・・物理ドライブ
* VG(Volume Group)・・・論理ドライブ、あるいは仮想ドライブ
* LV(Logical Volume)・・・論理パーティション、あるいは仮想パーティション

PVは単数でも複数でもいいけど、とにかくそれをひとまとめにして一つのVGを作る。そのVGをパーティション分けしたものがLV。

現状ではLVに32GB分割り当てられてて、残りの32GBは未割り当て、みたいな感じ。

Ubuntuインストール時にLVM用にフォーマットされたっぽいメッセージが出ていたのでLVM的なシステムはもうできあがっているのだろう。じゃあどこまで設定しなくてよくて、どこからしないといけないのか。

## LVMが作られたときのドライブの増やし方

参考：[How to Extend/Reduce LVM's (Logical Volume Management) in Linux - Part II](https://www.tecmint.com/extend-and-reduce-lvms-in-linux/)

要はLVMが問題になってくるのはドライブ容量を増やしたり減らしたりしたいときなので、そんなときにどうするのかを見てみる。

特に増やす場合。

1. 物理ドライブを差し込む。

2. 新たな物理ドライブに対して`sudo fdisk (デバイスパス:/dev/sdbみたいな)`でパーティションを作成。

   小分けにするつもりがなくても、全体をカバーするような大きな1つのパーティションを作る必要がある。

3. パーティション作成途中（具体的にはファイルシステム指定のところ）で、「そのパーティションはLVM用だよ」という指定をする。

   ここで、作成したパーティションにボリューム名（`/dev/sdb1`のようなもの）が付けられる。

4. 作成したパーティションに対して、`sudo pvcreate (ボリューム名)`でPVを作成。

   `sudo pvs`または`sudo pvdisplay`で確認可能。

5. `sudo vgextend (VG名) (ボリューム名＝PV名)`で、既存のVGに新たなPVを追加してVGを拡張する。

   `sudo vgs`で確認可能。

6. `sudo lvextend -l (拡張サイズ) (LVパス)`でLVを拡張。

7. `sudo resize2fs (LV名)`でファイルシステムも拡張。

   `sudo lvdisplay`、または`sudo lvs`で確認可能。

ややこしいけど、一つポイントとしては

* この流れでとにかく一番デカイのはVG

を押さえておくこと。

物理ドライブPVをいくつか集めてVGを作る。そのVGを切り分けていくつかのLVを作る。PVが増えたらVGも増やせるよね。LVが一つしか必要無いんだったらVG内に目一杯広げてもいいよね。ということ。

ただしLVを増やしたらファイルシステムも拡張してあげないといけないところはあまり感覚的では無い気がする。まあそんなもんだと思えばいいけど。

## 問題解消に向けて

ここではLVM管理はすでに作られていて、PV、VGもあるんだけど、LVがVGの半分に設定されてしまっている。何か意図はあるんだろうけど、今の自分にはいらない。

ということで上記の手順の6番から行う。ただし、拡張サイズを正確に調べないといけないのでそこから始める。

~~~shell
$ sudo lvdisplay
  --- Logical volume ---
  LV Path                /dev/ubuntu-vg/ubuntu-lv
  LV Name                ubuntu-lv
  VG Name                ubuntu-vg
  LV UUID                JJnp5s-c8S4-og9C-eYah-CTVs-8qbo-pfYajq
  LV Write Access        read/write
  LV Creation host, time ubuntu-server, 2021-05-04 19:04:58 +0900
  LV Status              available
  # open                 1
  LV Size                <28.06 GiB
  Current LE             7183
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
~~~

LVパスは`/dev/ubuntu-vg/ubuntu-lv`、使用しているサイズは`7183`。単位は知らない。

~~~shell
$ sudo vgdisplay
  --- Volume group ---
  VG Name               ubuntu-vg
  System ID
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  2
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                1
  Open LV               1
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               56.11 GiB
  PE Size               4.00 MiB
  Total PE              14365
  Alloc PE / Size       7183 / <28.06 GiB
  Free  PE / Size       7182 / 28.05 GiB
  VG UUID               fU2dSl-DSwh-thOK-vpDT-SJPK-8n18-A9vdH5
~~~

VG名は`ubuntu-vg`、使用しているサイズは`7183`（＝LVと同じ）、空いているサイズは`7182`。まさにこの`7182`、つまり`Free PE`の値が今知りたい数字。

よって、前項の手順6として以下を打つ。

~~~shell
$ sudo lvextend -l +7182 /dev/ubuntu-vg/ubuntu-lv
  Size of logical volume ubuntu-vg/ubuntu-lv changed from <28.06 GiB (7183 extents) to 56.11 GiB (14365 extents).
  Logical volume ubuntu-vg/ubuntu-lv successfully resized.
~~~

次に手順7として以下を打つ。

~~~shell
$ sudo resize2fs /dev/ubuntu-vg/ubuntu-lv
resize2fs 1.45.5 (07-Jan-2020)
Filesystem at /dev/ubuntu-vg/ubuntu-lv is mounted on /; on-line resizing required
old_desc_blocks = 4, new_desc_blocks = 8
The filesystem on /dev/ubuntu-vg/ubuntu-lv is now 14709760 (4k) blocks long.
~~~

この`on-line resizing required`というのは、つまりシステムを止めずに容量が拡張できたよということ。これをしたいがためのLVMらしい。

## 確認

~~~shell
$ sudo df -h
Filesystem                         Size  Used Avail Use% Mounted on
udev                               1.8G     0  1.8G   0% /dev
tmpfs                              378M  1.5M  376M   1% /run
/dev/mapper/ubuntu--vg-ubuntu--lv   56G  6.0G   47G  12% /
tmpfs                              1.9G     0  1.9G   0% /dev/shm
tmpfs                              5.0M     0  5.0M   0% /run/lock
tmpfs                              1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mmcblk0p2                     976M  106M  804M  12% /boot
/dev/loop0                          56M   56M     0 100% /snap/core18/1997
/dev/mmcblk0p1                     511M  7.9M  504M   2% /boot/efi
/dev/loop2                          56M   56M     0 100% /snap/core18/1944
/dev/loop1                          70M   70M     0 100% /snap/lxd/19188
/dev/loop3                          32M   32M     0 100% /snap/snapd/10707
/dev/loop4                          71M   71M     0 100% /snap/lxd/19647
/dev/loop5                          33M   33M     0 100% /snap/snapd/11588
tmpfs                              378M     0  378M   0% /run/user/1000
~~~

`Mounted on`のところが`/`になっているところ、これがUbuntuにとってのルートディレクトリであり、これの`Avail`が47Gとなっていることから、無事に拡張されたことが分かる。