# VMwareの概要

我らが[ArchWiki](https://wiki.archlinux.jp/index.php/VMware)を見てみても訳がわからないのでざっくりまとめ。

## 前提

* UEFIでCPUの仮想化を有効にする必要がある。これはマザーボードによってやり方が違う。自分の場合はUEFIを開いて「Advanced」→「CPU Configuration」の中に「Intel Virtualization Technology」っていうのがあったけど、既に有効化されていた。

## インストールまで

* VMwareを使いたい場合、VMware Workstation Proと同Playerという2種類あって、個人で無償で使う場合はPlayerの方を使う。Proは有償。
* PlayerのDLはここ（Proとの比較もある）→[VMware Workstation Player | VMware | JP](https://www.vmware.com/jp/products/workstation-player.html)
* またはターミナル上で`sudo pacman -S vmware-workstation`でもインストールできるっぽい。（事前に`pacman -Syu`やスナップショットを撮るなどすること）

### 仮想マシンの作成

* VMware Workstation Player（以下Player）を開いた後に新規仮想マシンを作成するか既存の仮想マシンを開くか聞かれる。
* 新規仮想マシンを作成する場合、まず空の仮想ハードディスクができて、そこにOSをインストールすることになる。
  * どうせインストールするなら最初から聞いといてやろか、ということでPlayerからOSをインストールするためのディスクの指定を要求される。
  * OSのインストールディスクは物理ドライブからでも、ISOイメージファイルからでもいいらしい。
  * 仮想ハードディスクはサイズが指定できる。
  * その他、割り当てる物理ハードウェアを細かくカスタマイズできる。
* 既存の仮想マシンはそれ用のファイルがあるはずなのでそれを開く。

## 参考

[仮想環境のインストールでエラー。ASRock X470のAMD-v設定はここ！ | ぺらラボ](https://pera-lab.com/archives/2087)
[Windows 10で様々なOSとWindows 11を楽しむ【VMware Workstation Player】 - PCまなぶ](https://pcmanabu.com/vmware-workstation-player/)
[VMware - ArchWiki](https://wiki.archlinux.jp/index.php/VMware)
[VMPlayerにCentOS6.9をインストールする - Qiita](https://qiita.com/347lionz/items/b98d957ce13768b9114b)
[今更ながらCentOS6.10をVMwareで構築 | 名古屋でシステム開発WEB制作なら | トランソニックソフトウェア](https://trans-it.net/centos610-vmware/)
[【2021年最新】CentOS7をVMware Player上にインストールしてみる！～画像付きで手順説明 | トリオス](https://trios.pro/centos7-vmware-player-install/)
