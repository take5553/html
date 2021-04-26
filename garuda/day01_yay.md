# 1日目　`yay`を入れてAUR(Arch User Repository)の使い勝手を良くする。

`yay`はAURヘルパーというもので、AURからパッケージをDL・ビルドするのを簡単にしてくれる。

## AUR(Arch User Repository)

あるソフトが公式リポジトリに採用されるためには、まずAURに登録しユーザーが試しに使ってみて、ちゃんと動くかどうかとか、便利かどうかなんかを見て投票し、得票数が良かったものが公式に採用される仕組み。

まあつまり、ユーザーの投稿によって成り立つリポジトリ。玉石混交。

悪意のあるものもひょっとしたらあるけど、悪意がある人はそもそもArch Linux上で悪さをしようとは思わないんじゃないか？

ちなみにもうすでにトラブルシューティングで`noto-color-emoji-fontconfig`をAURからDLしてインストールしている。この手順を自動化してくれて、さらに管理までしてくれるのが`yay`。

## `yay`のインストール

`yay`自体がAUR上にあるので、自分でDLしてビルドする必要がある。手動でAURからDLしてくるディレクトリを`~/aur`として作っているのでそこで作業をする。

~~~shell
$ cd ~/aur
$ git clone https://aur.archlinux.org/yay.git
$ cd yay/
$ makepkg -si
~~~

## `yay`の使い方

ほぼ`pacman`と同じ。ただし、`sudo`は付けない方がいいらしい。

参考
[yayの導入 \- Qiita](https://qiita.com/yamad_linuxer/items/11f114a22f73e3aab502)
[Arch: Manjaro で「pacman」と「yay」のコマンド操作〈H53〉 \- Linux あれこれ](https://furuya7.hatenablog.com/entry/2020/05/06/180426)