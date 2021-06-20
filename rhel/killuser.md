# ユーザーを強制的にログアウトする

何が原因なのかは不明だけど、RHELにSSH接続したまま一定時間放置するとキー入力を受け付けなくなる。でもログイン状態は続いているので、再度ログインしてハングアップしているユーザーを強制ログアウトさせる。

## 環境

RHEL 8

## 手順

ログインしているユーザーを確認

~~~shell
$ sudo w
14:31:28 up  1:27,  3 users,  load average: 0.04, 0.03, 0.03
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
takeshi  pts/0    192.168.1.5      13:05    1:13m  0.05s  0.05s -bash
takeshi  pts/1    192.168.1.5      14:31    6.00s  0.04s  0.02s sshd: takeshi [priv]
takeshi  tty2     tty2             13:16    1:27m  9.35s  0.09s /usr/libexec/tracker-miner-apps
~~~

ログイン時間から推測するに、`pts/0`が息をしていないユーザーと判明。

ログインとは結局サーバーがクライアントの相手をしてあげているということなので、相手をしてあげているプロセスは一体何なのかを調べる。

~~~shell
$ sudo ps -ef | grep pts/0
takeshi     2189    2160  0 13:05 ?        00:00:00 sshd: takeshi@pts/0
takeshi     2191    2189  0 13:05 pts/0    00:00:00 -bash
takeshi    13683   13524  0 14:32 pts/1    00:00:00 grep --color=auto pts/0
~~~

`2189`がPIDらしいので、これを`kill`する。

~~~shell
$ sudo kill -9 2189
~~~

## 参考

[ログインユーザを強制ログアウト](http://www.mikitechnica.com/12-logout.html)
[プロセスを終了するkillコマンドの使い方まとめ！【Linuxコマンド集】](https://eng-entrance.com/linux-command-kill)

