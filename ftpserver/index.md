# FTPサーバー構築

詳しい解説はありふれているので、最短手順だけメモ。

## 環境

サーバー：Raspberry Pi

ネットワーク：ローカル（セキュリティは無視）

## 手順

1. Raspberry PiにSSHでログイン

2. FTPサーバーインストール

   ~~~shell
   $ sudo apt -y install vsftpd
   ~~~

3. デフォルト設定ではユーザーごとにホームディレクトリにchrootされるので、それで良ければ何も変更するところがない。

4. ユーザー作成＆パスワード設定

   ~~~shell
   $ sudo useradd -m (ユーザー名)
   $ sudo passwd (ユーザー名)
   ~~~

## 動作確認

シェルで以下を打つ。

~~~shell
$ ftp (ユーザー名)@(Raspberry PiのIP)
~~~

パスワードを求められるので、入力したらログインできる。
