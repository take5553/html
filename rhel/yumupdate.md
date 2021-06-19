# OSのアップデート

## 環境

RHEL 8

## 手順

チェック。（Debian系でいう`apt update`）

~~~shell
$ sudo dnf check-update
~~~

アップグレード。（Debian系でいう`apt upgrade`）

~~~shell
$ sudo dnf upgrade
~~~

