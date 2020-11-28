# SSH接続

SSH接続について簡単に説明する。詳しくは「SSHとは」で検索。

## 概要

Raspberry Piを遠隔操作する方法の一種。

[初期設定の記事](startup1.html)でリモートデスクトップ接続をしたけど、あれも遠隔操作。SSH接続ではざっくり言うとRaspberry Piのデスクトップを介さず直接コンソール画面にログインするようなイメージ。

## 環境

- Windows10
- PowerShell 5.1

## 接続方法

最近のPowerShell（またはWindows）は標準でOpenSSHというソフトがインストールされているので、特に何をしなくてもPowerShell上で`ssh`コマンドが使える。

~~~shell
> ssh pi@192.168.1.201
~~~

これでRaspberry Piのコンソールに直接ログインできる。

以下書式。

~~~shell
> ssh (ユーザー名)@(Raspberry PiのIP)
~~~

OpenSSHは別にRaspberry Pi向けに作られているわけではないので、SSH接続を許可しているコンピューターなら何でもいい。

## ログアウト

ログインした状態からログアウトするには

~~~shell
$ exit
~~~

これでログアウトできる。ログアウトするだけでRaspberry Piの電源を切るわけではないので注意。

## 以後のRaspberry Piへのログイン

* リモートデスクトップ接続してからターミナルを立ち上げる
* SSH接続でログインする

デスクトップ操作は前提としないのでどちらでも可とするけど、主にはSSH接続かな。