# 4日目　Raspberry PiからWebサーバーのドキュメントルートをDL

このサイトをホストしているRaspberry Piから、ドキュメントルートをGaruda Linuxの方にDLしてブログ執筆の準備を進める。

## 手順

[このサイトのアップロードにはGitを使っている](../webserver/syncgit.html)ので、DLするのも`git clone`で行う。

まずGaruda LinuxからGitを使ってRaspberry Piにアクセスできるようにするために、`~/.ssh`に`config`というファイルを作成する。

~~~shell
$ micro ~/.ssh/config
~~~

以下を打つ。

```
Host raspberrypi
  User upload
  HostName (Raspberry PiのIP)
  Port (設定したSSHのポート番号)
  IdentityFile (uploadユーザーの秘密鍵の場所)
  IdentitiesOnly yes
```

そしてKonsoleで、ホームディレクトリに居ることを確認してから以下を打つ。

~~~shell
$ git clone raspberrypi:/home/takeshi/www/html/.git
~~~

そうするとホームディレクトリに`html`というディレクトリが作成されその中にドキュメントルートの内容が全てDLされている。

## 同期体制

メインPCで編集をしてRaspberry Piに`push`したとして、それをGaruda Linuxの方で`pull`しないと同期しない。

メインPCの方で[以前に作ったスクリプト](../webserver/syncgit.html)を実行。

その後にGaruda Linuxの方で以下を実行。

~~~shell
$ git pull origin master
~~~

そうするとGaruda Linuxの方にも反映された。

逆にGaruda Linuxの方で更新をして、Raspberry Piに`push`する。

