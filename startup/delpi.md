# 新しいユーザー`takeshi`を追加し`pi`ユーザーを削除

「Raspberry Piには`pi`ユーザーが存在する」というのは世界中で知られており、セキュリティ的にはあまりよろしくない。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 1.4

## 方法

### 新しいユーザーを作成

`pi`ユーザーでRaspberry Piにログインし、コンソールに以下を打ち込む。

```shell
$ sudo adduser takeshi
```

名前を変える場合は`takeshi`の部分を変える。

コマンドを打つと、「新しいユーザー用のパスワード」「パスワード確認」を求められた後、「Full Name[]」というようなものがしばらく聞かれるが、特に入力する必要は無い。最後に`Y`と入力してEnterを押せば作成完了。

### 新しいユーザーに権限付与

```shell
$ sudo usermod -G pi,adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input,spi,gpio takeshi
```

`pi`から`gpio`までスペースを含めてはいけない。

要は`pi`ユーザーが所属するグループ全部に、新しいユーザーも含めてやる。`pi`グループにも含めるのはやりすぎ感があるが、まあ一応。

`pi`ユーザーが所属するグループは以下のコマンドで確認できる。

```shell
$ groups pi
```

それぞれのグループの役割は[ここ](http://www.apnorton.com/blog/2017/11/25/Raspberry-Pi-Default-Groups/index.html)を参照。

### （任意）`pi`ユーザー内のファイルをごっそり新規ユーザーにコピー

```shell
$ sudo cp -r /home/pi/* /home/takeshi
```

`/home/pi/*`はpiユーザーのホームディレクトリにあるもの全部をコピー元とし、`/home/takeshi`でそのコピー先を指定。

正直`pi`ユーザーで特に何もしていないならコピーする必要は無いと思う。自分で作った必要なファイルは自分で退避させる。

### `pi`ユーザーの自動ログインを無効化＆新しいユーザーを自動ログインさせる

[前回記事](startup1.html)で設定したことと同じ内容なので、すでにしている人は飛ばして可。

```shell
$ sudo nano /etc/lightdm/lightdm.conf
```

126行目にある`autologin-user=pi`をコメントアウトする。

その後

```shell
$ sudo nano /etc/systemd/system/autologin@.service
```

として、28行目の`--autologin pi`を`--autorogin takeshi`とする。

また、ブートをデスクトップのままにしているとどうしても`pi`ユーザーがログインしてしまう。ブート後CLIモードで立ち上げるように設定する。

```shell
$ sudo raspi-config
```

`3 Boot Options`→`B1 Desktop / CLI`で`B1 Console`を選ぶ。

こうすると、Raspberry PiにHDMIケーブルをつないで立ち上げてもCLIしか表示されなくなる。その場合、`startx`コマンドを打てばGUIが立ち上がる。

### （任意）新しいユーザーで実際にパスワードログイン

できるかどうかの確認。

一旦Raspberry Piからログアウト。

```shell
$ exit
```

WindowsのPowerShellを起動し以下を打つ。

```shell
> ssh takeshi@192.168.1.201
takeshi@192.168.1.201's password:
Linux takeshipi 5.4.51-v7+ #1333 SMP Mon Aug 10 16:45:19 BST 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Linux takeshipi 5.4.51-v7+ #1333 SMP Mon Aug 10 16:45:19 BST 2020 armv7l

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Sun Sep 27 22:49:38 2020 from 192.168.1.5
takeshi@takeshipi:~ $
```

IPは各自の設定に置き換えること。パスワードを要求されるので、入力したらログインできるはず。

### 新しいユーザーのホームディレクトリに`.ssh`ディレクトリを作成

```shell
takeshi@takeshipi:~ $ mkdir /home/takeshi/.ssh
```

一旦ログアウト。

~~~shell
takeshi@takeshipi:~ $ exit
logout
Connection to 192.168.1.201 closed.
~~~

### 新規ユーザーがパスワード無しで`sudo`コマンドを実行できるようにする

以下、`pi`ユーザー。

```shell
$ sudo visudo
```

最後に`takeshi ALL=(ALL) NOPASSWD: ALL`の一文を加えて保存終了をする。保存は`ctrl + S`、終了は`ctrl + X`。

### PC上で秘密鍵と公開鍵を作成

これはRaspberry Pi上ではなく、PC上で行う。

```shell
> ssh-keygen -t rsa
```

打った後、以下を聞かれる。

- 鍵の保存場所のフルパス

  デフォルトとされているのは`c:/Users/(Windowsのユーザー名)/.ssh/id_rsa`で、ここに保存すると後々`ssh`コマンドの入力が楽になるが、秘密鍵の保存場所のデフォルトなので誰かにPCを乗っ取られたらまず間違いなく特定される。

  ちなみに`id_rsa`は拡張子無しのファイル名であり、`c:/Users/(Windowsのユーザー名)/.ssh`フォルダに`id_rsa`ファイルと`id_rsa.pub`ファイルが作成される。この`c:/Users/(windowsのユーザー名)/.ssh`フォルダは事前に作っておく必要有り。

- パスフレーズ設定、再確認

  もし万が一秘密鍵が盗まれても、パスフレーズを設定しておけばそれが分からない限り秘密鍵を使われることはない。ただ、秘密鍵を使う度にパスフレーズの入力を求められるので、利便性は少し失われる。逆にパスフレーズを設定しなければセキュリティは下がるけど、入力を求められないので利便性は上がる。

ちなみに以前に作ったことがあればそれを再利用することが可能。

### 公開鍵をRaspberry Piに転送

PowerShell上で行う。

```shell
> scp c:/Users/(Windowsのユーザー名)/.ssh/id_rsa.pub takeshi@192.168.1.201:~/.ssh
```

ログインパスワードを求められるので入力すると`id_rsa.pub`が転送される。秘密鍵を別の場所に保存した人は転送元の指定を変更すること。

### 公開鍵を`authorized_keys`にリネーム

再度Raspberry Piに新しいユーザーでログインして以下を打つ。

```shell
takeshi@takeshipi:~ $ mv ~/.ssh/id_rsa.pub ~/.ssh/authorized_keys
```

これで、`id_rsa.pub`が`authorized_keys`という名前にリネームされる。

### パーミッション設定

```shell
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```

これをしないと公開鍵認証が動かない。

### 新しいユーザーで公開鍵認証ログイン

一旦Raspberry Piからログアウトし、PowerShellから以下を打つ。

```shell
> ssh takeshi@192.168.1.201
```

ログインコマンドとしては同じだが、`newuser`のパスワードは聞かれない。秘密鍵にパスフレーズを設定していたらそっちを聞かれる。

秘密鍵をデフォルト以外の場所に保存した人は、`-i`オプションで保存場所を指定すること。絶対パスか、カレントフォルダからの相対パス。逆にデフォルトの人は`-i`オプションそのものがいらない。

### piユーザーの削除

```shell
takeshi@takeshipi:~ $ sudo userdel -r pi
```

確認コマンドは以下。

```shell
$ id -a pi
id: ‘pi’: no such user 
```

