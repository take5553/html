# パッケージマネージャー

パッケージマネージャーとは、Raspberry Piでいう`apt`のことで、Garuda Linuxでは`pacman`だそうだ。ここでは`pacman`の使い方をゆるくまとめていく。

基本的にRaspberry Piと比較（つまりDebian系と比較）していく。

## パッケージをアップグレード

~~~shell
$ sudo pacman -Syu
~~~

ラズパイ：`$ sudo apt update && sudo apt upgrade`

その後ろの`y`は必ず`S`の後に付けないといけない。`-Sy`で`/etc/pacman.conf`内に書かれているリポジトリを更新、続けて`u`でアップグレード。つまり全部最新になるってこと。

## リポジトリを最新にする

~~~shell
$ sudo pacman -Syy
~~~

ラズパイ：`$ sudo apt update`

リポジトリ情報を最新にするだけならこっち。でもArch Linux（とGaruda Linux）では何かソフトを入れる度にリポジトリだけじゃなくてパッケージのアップグレードもすることが推奨されているらしい。

## パッケージをインストール

~~~shell
$ sudo pacman -S (パッケージ名)
~~~

ラズパイ：`$ sudo apt install (パッケージ名)`

`-S`または`--sync`。

## パッケージをインストール（強制Yes）

~~~shell
$ sudo pacman -S --noconfirm (パッケージ名)
~~~

ラズパイ：`$ sudo apt install -y (パッケージ名)`

`--noconfirm`は省略できない模様。

## パッケージのアンインストール

~~~shell
$ sudo pacman -R (パッケージ名)
~~~

ラズパイ：`$ sudo apt remove (パッケージ名)`

## パッケージのアンインストール（依存関係も含む）

~~~shell
$ sudo pacman -Rs (パッケージ名)
~~~

ラズパイ：`$ sudo apt remove --purge (パッケージ名)`

ただし挙動は少し違ってて、`apt reove --purge`は完全削除なのに対して、`pacman -Rs`は依存関係の中で使われていないものをチョイスして削除するらしい。

## パッケージ情報の検索

~~~shell
$ sudo pacman -Si (パッケージ名)
~~~

ラズパイ：`$ sudo apt show (パッケージ名)`

`pacman -S (パッケージ名)`でインストールされるパッケージの情報を表示する。自分的には地味に必須。

## パッケージ情報の検索（部分一致）

~~~shell
$ sudo pacman -Ss (パッケージ名)
~~~

ラズパイ：`$ sudo apt search (パッケージ名)`

複数のリポジトリから引っかかるものを検索。

## その他

[Pacman/比較表 \- ArchWiki](https://wiki.archlinux.jp/index.php/Pacman/%E6%AF%94%E8%BC%83%E8%A1%A8)

大体載ってる。