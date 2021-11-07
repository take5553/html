# Ubuntu Desktop編

メインPCのOSとして導入する。

もうLinuxを導入するのは何回目か分からん。ほぼ備忘録。

## インストールUSB作成

### ISOファイルを取ってくる

[Download Ubuntu Desktop | Download | Ubuntu](https://ubuntu.com/download/desktop)

### USBに書き込み

※前提：Linux上でのコマンド

~~~shell
$ sudo dd of=/dev/sd(挿したUSBメモリの認識アルファベット) bs=1M status=progress if=(Ubuntuのisoファイル)
~~~

挿したUSBメモリの認識アルファベットはつまり自分の場合でいえば`e`になる。

`sudo dmesg`で直近のメッセージに残ってるやつを見ればいい。

### USB取り出し

~~~shell
$ sudo eject /dev/se(挿したUSBメモリの認識アルファベット)
~~~

