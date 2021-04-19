# Xrdpをインストール（※失敗）

Raspberry Piでやったように、WindowsからリモートデスクトップでGaruda Linuxを操作できるようにしてみる。

いや、別に必要は無いんだけど、記事書く用のスクリーンショットが撮りにくいんですよ。

※先に結果を書くと、上手く動かず失敗。

## 手順1

### AUR用のディレクトリを作成

AURとは[Arch User Repository](https://wiki.archlinux.jp/index.php/Arch_User_Repository)のことで、公式じゃないけどユーザーが「これはいいぞ」ということで追加されたリポジトリのこと。ここからDLするパッケージはちょっと特殊な手順を踏む必要がある。

なので、専用のディレクトリを作成する。

~~~shell
$ mkdir ~/aur
$ cd ~/aur
~~~

### xrdpのPKGBUILDアーカイブをDL

~~~shell
$ git clone https://aur.archlinux.org/xrdp.git
~~~

### `PKGBUILD`と`.install`が正しいかチェック

~~~shell
$ cd xrdp
$ micro PKGBUILD
$ micro xrdp.install
~~~

中身を開いて正しいコマンドが書かれているか確認しろとのこと。でもこんなん分かるわけないやん。

### ビルド＆インストール

~~~shell
$ makepkg -si
~~~

通常ユーザーで実行しろとのこと。

### Xrdpを有効化&起動

~~~shell
$ sudo systemctl enable -now xrdp xrdp-sesman
~~~

### 結果

だめでした。上手く起動しない。

## 手順2

### xorgxrdp-gitをインストール

~~~shell
$ cd ~/aur
$ git clone https://aur.archlinux.org/xorgxrdp-git.git
$ cd xorgxrdp-git
$ makepkg -si
~~~

### 設定

~~~shell
$ sudo micro /etc/X11/Xwrapper.config
~~~

以下を記入。

~~~
allowed_users=anybody
~~~

### Xrdpを再起動

~~~shell
$ sudo systemctl restart xrdp xrdp-sesman
~~~

### 結果

ダメでした。ロード画面っぽいのは表示されたけど、画面が真っ黒のまま動かず。

## 後片付け

~~~shell
$ sudo systemctl stop xrdp xrdp-sesman
$ sudo systemctl disable xrdp xrdp-sesman
$ sudo pacman -Rs xrdp xorgxrdp-git
~~~

## 参考

[Xrdp \- ArchWiki](https://wiki.archlinux.jp/index.php/Xrdp)
[Arch User Repository \- ArchWiki](https://wiki.archlinux.jp/index.php/Arch_User_Repository)