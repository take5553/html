# SSHサーバーの起動

最初上手くいかなかったので備忘録として。

## 環境

RHEL 8

## 手順

### インストール

インストールしていなかったらインストールする。

~~~shell
$ sudo yum install openssh-server
~~~

### コマンド

起動

~~~shell
$ sudo systemctl start sshd
~~~

停止

~~~shell
$ sudo systemctl stop sshd
~~~

再起動

~~~shell
$ sudo systemctl restart sshd
~~~

自動起動

~~~shell
$ sudo systemctl enable sshd
~~~

自動起動を解除

~~~shell
$ sudo systemctl disable sshd
~~~

## 接続

ファイアーウォールで防がれていないか確認。

~~~shell
$ sudo firewall-cmd --list-all
~~~

`ssh`という文字があったらポートが開いている。

もし無かったら

~~~shell
$ sudo firewall-cmd --permanent --add-service=ssh
~~~

と打つ。

## 参考

[【初心者向け】SSHのインストールと設定方法](https://eng-entrance.com/linux-ssh-install)

