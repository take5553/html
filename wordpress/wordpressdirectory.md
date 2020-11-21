# ドキュメントルート内にWordPress用のディレクトリを作成

ドキュメントルートを以下の構成にしたい。

~~~
ドキュメントルート
　├.git
　├startup
　│　└ファイル色々
　├webserver
　│　└ファイル色々
　├wordpress
　│　└ファイル色々
　└wordpressblog
　　 └wordpressの実行ファイル等
~~~

「じゃあそうしろよ」って感じだけど、問題は`wordpressblog`ディレクトリはGit管理したくないということ。

Git管理から外すということで`.gitignore`に書いていくけど、ローカルでは`wordpressblog`フォルダを一切作らないというところでちょっと実験。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
  - Git version 2.28.0.windows.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Git version 2.20.1

## 方法

まずリモートにログインし、

~~~shell
$ cd www/html
$ mkdir wordpressblog
$ chown :upload wordpressblog
$ cd wordpressblog
$ nano test.txt
~~~

として`test.txt`の中身を適当に書いて保存。ローカルからのPushでこれが消えなければよい。

ローカル側の`.gitignore`で以下を追記。

~~~
wordpressblog/
~~~

保存してPush。Pushには[前回](../webserver/syncgit.html)のスクリプトを使用した。

Push後、再度Raspberry Piにログインして確認。

~~~shell
$ cd www/html
$ ls
index.html  startup  webserver  wordpress  wordpressblog
$ cd wordpressblog
$ ls
test.txt
$ cat test.txt
asdf
~~~

よし。

