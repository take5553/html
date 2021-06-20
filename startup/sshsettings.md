# SSHの設定（rootログイン禁止、ポート変更、パスワード認証禁止等）

セキュリティの観点からSSHの設定をしていく。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 1.4

## 方法

基本的には`/etc/ssh/sshd_config`というファイルを編集することになる。このファイルを開くコマンドは以下。

```shell
$ sudo nano /etc/ssh/sshd_config
```

これ以降は、上記のファイルを編集するということを前提にして説明していく。ちなみに行頭に`#`が付いてる行で、`#`の直後がスペースじゃないものはデフォルト値。

### rootログイン禁止

```
#PermitRootLogin prohibit-password
↓
PermitRootLogin no
```

公開鍵を設定していない`root`で、パスワードログインを禁止していたらそりゃできないでしょってことなんだけど、一応`no`として完全禁止にする。

### （任意）ポート変更

```
#Port 22
↓
Port （好きな番号）
```

いろいろ予約されているポート番号があるので、49152 ～ 65535の間から選ぶようにする。

22番ポートというのはSSH用のポートとしてあまりにも有名だからハッキングを受けやすく、ほとんどの場合は変更するということになっている。

が、Raspberry Piを運用する限りではこのポート変更はあまり意味がない。詳しい説明は次回の記事に書く。

### パスワードログインの無効化

```
#PasswordAuthentication yes
↓
PasswordAuthentication no
```

パスワードログインを有効にしているとでたらめにパスワードを入れられてひょっとしたら破られるかもしれないから。これは秘密鍵＆公開鍵でログインできることを確認してからでないとログインできなくなる。（その場合はモニターをつないで直接ログインするようにすればRSA鍵が必要ないので何とかなるかも）

### その他の設定

せっかくなので他の項目も知りたいという人は「sshd_config 解説」とかでググろう。最初は必要そうな項目について全部書こうかと思ったけど、結局初期値のままで良かったり、実際に使う人が自分しかいないのであれば必要ない設定項目ばっかりだった。

一応このページの最後に追加の設定を書いておくので、もっとセキュアにしたい人はそちらを参照。

### 設定ファイルの保存＆終了

`ctrl + S`で保存、`ctrl + X`で終了。

### SSHの再起動

設定の再読み込みのためSSHの再起動。

```shell
$ sudo /etc/init.d/ssh restart
```

ログインを試してみるため、ログアウトがてらRaspberry Pi自体を再起動。

```shell
$ sudo reboot
```

接続が切れるので、PowerShellで以下を打つ。

```shell
> ssh takeshi@192.168.1.201
```

ちゃんと設定できていればログインできるはず。ポート番号を変えた人は`-p`オプションが必要。秘密鍵の場所がデフォルトじゃない人は`-i`オプションで秘密鍵の指定も必要。

## よりセキュアな設定

~~~
#SyslogFacility AUTH
↓
SyslogFacility AUTHPRIV

#LogLevel INFO
↓
LogLevel VERBOSE

#AllowTcpForwarding yes
↓
AllowTcpForwarding no

#X11Forwarding yes
↓
X11Forwarding no
~~~

