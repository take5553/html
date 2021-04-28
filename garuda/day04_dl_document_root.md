# 4日目　Raspberry PiからWebサーバーのドキュメントルートをDL＆UL

このサイトをホストしているRaspberry Piから、ドキュメントルートをGaruda Linuxの方にDLしてブログ執筆の準備を進める。

ほぼGaruda Linux関係なし。

## 手順

[このサイトのアップロードにはGitを使っている](../webserver/syncgit.html)ので、DLするのも`git clone`で行う。

まずGaruda LinuxからGitを使ってRaspberry Piにアクセスできるようにするために、`~/.ssh`に`config`というファイルを作成する。

~~~shell
$ micro ~/.ssh/config
~~~

以下を記入して保存。

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

### Windows→Raspberry Pi→Garuda Linux

メインPCで編集をしてRaspberry Piに`push`したとして、それをGaruda Linuxの方で`pull`しないと同期しない。

メインPCの方で[以前に作ったスクリプト](../webserver/syncgit.html)を実行。

その後にGaruda Linuxの方で以下を実行。

~~~shell
$ git pull origin master
~~~

そうするとGaruda Linuxの方にも反映された。

### Garuda Linux →Raspberry Pi→Windows

逆にGaruda Linuxの方で更新をして、Raspberry Piに`push`する。

まずGaruda Linuxの方で`commit`するのは初めてなので、その前にメールアドレスと名前を登録しておく。大体誰もが最初に怒られるやつ。

~~~shell
$ git config --global user.email "(メアド)"
$ git config --global user.name "(名前)"
~~~

そして`commit`＆`push`

~~~shell
$ git add .
$ git commit -m "test on Garuda"
$ git push origin master
~~~

本来ならこれで終わりだけど、ウチのリモートリポジトリはベアじゃないのでもう一手間必要。この辺の詳しい解説は[こちら](../webserver/syncgit.html)。

Raspberry Piにログインしてから以下を打つ。

~~~shell
$ cd /home/takeshi/www/html
$ git reset --hard
~~~

これでRaspberry Pi上で反映された。

これをWindows上に反映させるには`pull`をすれば良い。PowerShellを開き、ローカルのドキュメントルートに移動して以下を打つ。

~~~shell
> git pull origin master
~~~

そうすると反映される。