# Pythonのバージョンを指定してインストール

ソースコードから無理やり突っ込んでしまう。

## 環境

RHEL 8

## 手順

必要なツールをインストール。

~~~shell
$ sudo yum groupinstall "development tools"
$ sudo yum install bzip2-devel gdbm-devel libffi-devel \
  libuuid-devel ncurses-devel openssl-devel readline-devel \
  sqlite-devel tk-devel wget xz-devel zlib-devel
~~~

次に[Download Python | Python.org](https://www.python.org/downloads/)から好みのバージョンの「Gzipped source tarball」のリンクをコピーする。ここではPython 3.9.5をDLすることとする。

RHELのコンソール上で以下を打ちソースコードをDL。

~~~shell
$ wget https://www.python.org/ftp/python/3.9.5/Python-3.9.5.tgz
~~~

解凍からのインストール。

~~~shell
$ tar -xzvf Python-3.9.5.tgz
$ cd Python-3.9.5
$ ./configure
$ make
$ sudo make install
~~~

確認。

~~~shell
$ python3 --version
Python 3.9.5
~~~

ちなみにインストール先は`/usr/local/bin`

## 参考

[CentOS 環境のPython: Python環境構築ガイド - python.jp](https://www.python.jp/install/centos/index.html)

