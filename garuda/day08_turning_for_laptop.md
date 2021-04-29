# 8日目　Laptop用設定を適用する

Garuda LinuxはデスクトップPC用に最適化されているものらしいので、ラップトップPC用に設定を変更する。こうすることで変にCPUを使って温度が上がりすぎたりするのを防いでくれるらしい。

参考：[[GUIDE] Configuring Garuda Linux for Laptop - Garuda Community - Garuda Linux Forum](https://forum.garudalinux.org/t/guide-configuring-garuda-linux-for-laptop/7685)

## 手順

### システムをアップデートしておく

「パックマン、シュー」

~~~shell
$ sudo pacman -Syu
(または)
$ yay -Syu
~~~

「念の為にアップデートしておく」だけではなく、`pacman -Syu`をするとシステムのスナップショットが撮られて、後でシステムが不安定になったとしたらアップデートする前に戻れる。だから記録を取る意味でもアップデートしておいた方が良い。

### `performance-tweaks`および`gamemode`の削除

~~~shell
$ pacman -R performance-tweaks gamemode
~~~

これはCPUをフル稼働させて、特にラップトップでは危険らしい。ちなみに`gamemode`は入っていないと怒られる場合は`performance-tweaks`のみで良い。

### Linuxカーネルの削除

マジで？

子曰く「Garuda LinuxにはLinux Zenが組み込まれておるはずじゃが、それはデスクトップ用じゃ。ラップトップには普通のLinux、または安定版のLinux LTSを入れるのじゃ。」

先にLinuxとLinux LTSを入れる。

~~~shell
$ sudo pacman -S linux linux-lts
...
エラー: ファイル 'linux-lts-5.10.32-1-x86_64.pkg.tar.zst' を ftp.jaist.ac.jp から取得するのに失敗しました : The requested URL returned error: 404
警告: 複数のファイルの取得に失敗しました
エラー: 処理を完了できませんでした (ファイルの取得に失敗しました)
エラーが発生したため、パッケージは更新されませんでした。
~~~

あれー？

これは日本のリポジトリがあかんのかな。追加しよか。

公式の[Arch Linux - Pacman Mirrorlist Generator](https://archlinux.org/mirrorlist/)で日本の別のリポジトリを調べることができる。

探してみると`Server = https://mirrors.cat.net/archlinux/$repo/os/$arch`が良さげなので、これを追加してみる。

~~~shell
$ sudo micro /etc/pacman.d/mirrorlist
~~~

サーバーを追記して保存終了。その後にパッケージリストをアップデート。ついでにシステムもアップデート。

~~~shell
$ sudo pacman -Syyu
~~~

`y`は2つ必要。

そして再度LinuxとLinux-LTSのインストールをインストールすると、今度は少し時間がかかったけど入った。

~~~shell
$ sudo pacman -S linux linux-lts
~~~

その後、Linux-Zenを削除する。

~~~shell
$ sudo pacman -R linux-zen
~~~

### `auto-cpufreq`および`thermald`のインストール

CPUをオーバーヒートから防ぐパッケージをインストールする。

~~~shell
$ sudo pacman -S auto-cpufreq thermald
~~~

インストールできたら有効化する。

~~~shell
$ sudo systemctl enable auto-cpufreq
$ sudo systemctl start auto-cpufreq
$ sudo systemctl enable thermald
$ sudo systemctl start thermald
~~~

`auto-cpufreq`は、CPUがある一定温度に達するとCPUの周波数（またはスピード）を勝手に制限してくれるらしい。`thermald`はCPUがある一定温度に達すると勝手に頑張ってファンとか回して温度を下げようとしてくれるらしい。

ある一定温度って何度やねん。

## 再起動

これで動かんかったらどーしよ、とドキドキしながら再起動したけど普通に動いた。

カーネルもさっきまでLinux-ZenだったのがLinux-LTSが動いているらしい。カーネルの入れ替えって結構簡単なのね。

![image-20210429231702360](image/day08_turning_for_laptop/image-20210429231702360.png)

