# SSHログイン＆セキュリティ設定

基本的にはSSH接続でRaspberry Piを運用するときの、最初にすべきセキュリティ設定をメモしておく。

## 環境

* ローカル（PC側）
  - Windows10
  - Powershell 5.1
* リモート（Raspberry Pi）
  * Raspberry Pi 3B+
  * Raspberry Pi OS 10.4

前提としてインストール時に「pi」ユーザーのパスワードは「raspberry」と違うものを設定し、OSのアップデートは済ませているものとする。

## 方針

* 自分用の新しいユーザーを作成し、`pi`ユーザーを削除する。

  →Raspberry Piには`pi`ユーザーが存在するというのは世界中で知られており、セキュリティ的にはあまりよろしくない。

* SSHの設定をし、パスワード認証ではログインできないようにする。

  →パスワードは破られる可能性があるので、公開鍵認証でしかログインできないようにする。

## 方法

### rootにパスワードを設定する

Raspberry Piのrootにはパスワードが設定されていない、とネットでよく見るけど実際のところpiユーザーから`su`コマンドを打ってもパスワード無し、piユーザーのパスワード共に通らずrootにはなれなかった。よく分からないけど別にパスワードを設定するのは手間ではないのでやっておく。

```shell
$ sudo passwd root
```

設定したいパスワードを入力すると、次からは`su`コマンドでrootユーザーになれる。

### 新しいユーザーを作成し、「pi」ユーザーを消去

これはいくつか手順を踏む。

#### newuser（新しいユーザー）作成

```shell
$ sudo adduser newuser
```

名前を変える場合は`newuser`の部分を変える。

コマンドを打つと、「newuser用のパスワード」「パスワード確認」を求められた後、「Full Name[]」というようなものがしばらく聞かれるが、特に入力する必要は無い。最後に`Y`と入力してEnterを押せば作成完了。

#### newuserに権限付与

```shell
$ sudo usermod -G pi,adm,dialout,cdrom,sudo,audio,video,plugdev,games,users,netdev,input,spi,gpio newuser
（piからgpioまでスペースを入れてはいけない）
```

要はpiユーザーが所属するグループ全部に、newuserも含めてやる。piグループにも含めるのはやりすぎ感があるが、まあ一応。

piユーザーが所属するグループは以下のコマンドで確認できる。

```shell
$ groups pi
```

それぞれのグループの役割は[ここ](http://www.apnorton.com/blog/2017/11/25/Raspberry-Pi-Default-Groups/index.html)を参照。

#### （任意）piユーザー内のファイルをごっそり新規ユーザーにコピー

```shell
$ sudo cp -r /home/pi/* /home/newuser
```

`/home/pi/*`はpiユーザーのホームディレクトリにあるもの全部をコピー元とし、`/home/newuser`でそのコピー先を指定。

正直piユーザーで特に何もしていないならコピーする必要は無いと思う。自分で作った必要なファイルは自分で退避させる。

#### piユーザーの自動ログインを無効化＆newuserを自動ログインさせる

[前回記事](startup.html)の最後で設定したことと同じ内容なので、すでにしている人は飛ばして可。

```shell
$ sudo nano /etc/lightdm/lightdm.conf
```

126行目にある`autologin-user=pi`をコメントアウトする。

その後

```shell
$ sudo nano /etc/systemd/system/autologin@.service
```

として、28行目の`--autologin pi`を`--autorogin newuser`とする。

また、ブートをデスクトップのままにしているとどうしてもpiユーザーがログインしてしまう。ブート後CLIモードで立ち上げるように設定する。

```shell
$ sudo raspi-config
```

`3 Boot Options`→`B1 Desktop / CLI`で`B1 Console`を選ぶ。

こうすると、RaspberryPiにHDMIケーブルをつないで立ち上げてもCLIしか表示されなくなる。その場合、`startx`コマンドを打てばGUIが立ち上がる。

#### （任意）newuserで実際にパスワードログイン

できるかどうかの確認。

一旦Raspberry Piからログアウト。

~~~shell
$ exit
~~~

そののち、WindowsのPowershellで以下を打つ。

~~~shell
（ローカル上）
$ ssh newuser@（ラズパイのIP）
~~~

パスワードを要求されるので、入力したらログインできるはず。

#### 新規ユーザーがパスワード無しで`sudo`コマンドを実行できるようにする

```shell
$ sudo visudo
```

最後に`newuser ALL=(ALL) NOPASSWD: ALL`の一文を加えて保存終了をする。保存は`ctrl + S`、終了は`ctrl + X`。

#### Raspberry Pi側に.sshディレクトリを作成

```shell
$ mkdir ~/.ssh
```

ログイン後カレントディレクトリを変更していないなら`~/`は無くても可。`~/`はそのユーザーのホームディレクトリを指す。

#### PC上で秘密鍵と公開鍵を作成

これはRaspberry Pi上ではなく、通常使うPC上で行う。

```shell
（ローカル上）
$ ssh-keygen -t rsa
```

打った後、以下を聞かれる。

* 鍵の保存場所のフルパス

  デフォルトとされているのは`c:/Users/(Windowsのユーザー名)/.ssh/id_rsa`で、ここに保存すると後々`ssh`コマンドの入力が楽になるが、秘密鍵の保存場所のデフォルトなので誰かにPCを乗っ取られたらまず間違いなく特定される。

  ちなみに`id_rsa`は拡張子無しのファイル名であり、`c:/Users/(Windowsのユーザー名)/.ssh`フォルダに`id_rsa`ファイルと`id_rsa.pub`ファイルが作成される。この`c:/Users/(windowsのユーザー名)/.ssh`フォルダは事前に作っておく必要有り。

* パスフレーズ設定、再確認

  もし万が一秘密鍵が盗まれても、パスフレーズを設定しておけばそれが分からない限り秘密鍵を使われることはない。ただ、秘密鍵を使う度にパスフレーズの入力を求められるので、利便性は少し失われる。逆にパスフレーズを設定しなければセキュリティは下がるけど、入力を求められないので利便性は上がる。

ちなみに以前に作ったことがあればそれを再利用することが可能。

#### 公開鍵をRaspberry Piに転送

```shell
（ローカル上）
$ scp c:/Users/(Windowsのユーザー名)/.ssh/id_rsa.pub newuser@（RaspberryPiのIP）:/home/newuser/.ssh
```

ログインパスワードを求められるので入力すると`id_rsa.pub`が転送される。秘密鍵を別の場所に保存した人は転送元の指定を変更すること。

#### 公開鍵を`authorized_keys`にリネーム

再度Raspberry Piにnewuserでログインして以下を打つ。

```shell
（ローカル上）
$ ssh newuser@(ラズパイのIP)

（リモート上）
$ mv ~/.ssh/id_rsa.pub >> ~/.ssh/authorized_keys
```

これで、`id_rsa.pub`が`authorized_keys`という名前にリネームされる。

#### パーミッション設定

```shell
$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys
```

これをしないと公開鍵認証が動かない。

#### newuserで公開鍵認証ログイン

一旦RaspberryPiからログアウトし、メインPCのコンソールから以下を打つ。

```shell
（ローカル上）
$ ssh newuser@（RaspberryPiのIP） -i （秘密鍵のパス）
```

ログインコマンドとしては同じだが、`newuser`のパスワードは聞かれない。秘密鍵にパスフレーズを設定していたらそっちを聞かれる。

秘密鍵をデフォルト以外の場所に保存した人は、`-i`オプションで保存場所を指定すること。絶対パスか、カレントフォルダからの相対パス。逆にデフォルトの人は`-i`オプションそのものがいらない。

#### piユーザーの削除

```shell
（リモート上）
$ sudo userdel -r pi
```

確認コマンドは以下。

```shell
$ id -a pi
id: ‘pi’: no such user 
```

### SSHの設定（rootログイン禁止、ポート変更、パスワード認証禁止等）

基本的には`/etc/ssh/sshd_config`というファイルを編集することになる。このファイルを開くコマンドは以下。

```shell
$ sudo nano /etc/ssh/sshd_config
```

これ以降は、上記のファイルを編集するということを前提にして説明していく。ちなみに行頭に`#`が付いてる行で、`#`の直後がスペースじゃないものはデフォルト値。

#### rootログイン禁止

```
#PermitRootLogin prohibit-password
↓
PermitRootLogin no
```

公開鍵を設定していないrootで、パスワードログインを禁止していたらそりゃできないでしょってことなんだけど、一応`no`として完全禁止にする。

#### （任意）ポート変更

```
#Port 22
↓
Port （好きな番号）
```

いろいろ予約されているポート番号があるので、49152 ～ 65535の間から選ぶようにする。

22番ポートというのはSSH用のポートとしてあまりにも有名だからハッキングを受けやすく、ほとんどの場合は変更するということになっている。が、Raspberry Piを運用する限りではこのポート変更はあまり意味がない。詳しい説明は次回の記事に書く。

#### パスワードログインの無効化

```
#PasswordAuthentication yes
↓
PasswordAuthentication no
```

パスワードログインを有効にしているとでたらめにパスワードを入れられてひょっとしたら破られるかもしれないから。これは秘密鍵＆公開鍵でログインできることを確認してからでないとログインできなくなる。（その場合はモニターをつないで直接ログインするようにすればRSA鍵が必要ないので何とかなるかも）

#### その他の設定

せっかくなので他の項目も知りたいという人は「sshd_config 解説」とかでググろう。最初は必要そうな項目について全部書こうかと思ったけど、結局初期値のままで良かったり、ユーザーが自分しかいないのであれば必要ない設定項目ばっかりだった。

#### 設定ファイルの保存＆終了

`ctrl + S`で保存、`ctrl + X`で終了。

#### SSHの再起動

```shell
$ sudo /etc/init.d/ssh restart
```

一旦ログアウトがてらにRaspberry Pi自体を再起動。

~~~shell
$ sudo reboot
~~~

メインPCから

```shell
$ ssh newuser@（RaspberryPiのIP） -p （ポート番号） -i （秘密鍵の場所）
```

とすればログインできるはず。ポート番号を変えていない人は`-p`オプションは不要。秘密鍵の場所がデフォルトの人は`-i`オプションも不要。

### 終了

お疲れさまでした。次回はルーターにポートフォワーディング（またはポート開放）を設定する。それまではルーターがファイアーウォールを持っているはずなので（多分）、まだ外部から侵入なんてことはないはず。

## 参考

[初心者向！Raspberry Pi 最低限のセキュリティ設定【所要時間 30分】 – Qiita](https://qiita.com/mochifuture/items/00ca8cdf74c170e3e6c6)
[ラズパイ入手したらまずやること、デフォルトユーザーとSSHログイン認証を変更する (1/3) – MONOist（モノイスト）](https://monoist.atmarkit.co.jp/mn/articles/1912/11/news022.html)
[ラズパイでやらなければいけない４つのセキュリティ対策！ – Qiita](https://qiita.com/nokonoko_1203/items/94a888444d5019f23a11)
[Raspberry Pi Webサーバー セキュリティ設定 | アラコキからの Raspberry Pi 電子工作](https://arakoki70.com/?p=2139)
[ラズパイ４でpiユーザのIDを変更する方法 | rs-techdev](https://rs-techdev.com/archives/32)
[[Raspberry Pi\] ラズパイ & Remmina on LinuxでRDP接続してリモートデスクトップさせてみる方法](https://www.tacoskingdom.com/blog/47)
[Raspberry Pi3のLAN外からのSSH接続設定方法 – Qiita](https://qiita.com/3no3_tw/items/4b5975a9f3087edf4e20)
[Raspberry Piに外部ネットワークからアクセスできる様にして携帯でペットを遠隔監視する方法 – Qiita](https://qiita.com/kinpira/items/c9e6dc910e8d96e8c19b)
[セキュアなSSHサーバの設定 – Qiita](https://qiita.com/comefigo/items/092137ac40f319cb14fa)
[RaspberryPi 3 デフォルトユーザpiの変更 | そう備忘録](https://www.souichi.club/raspberrypi/default-user-pi/#自動ログインユーザの変更)
[Raspberry PiのCLI/GUIログインの切り替え – ysdyt.net for tech memo](http://ysdyt.github.io/blog/2015/03/27/raspi-cli-gui-login/)

TCPフォワーディングについて
[sshポートフォワーディングの使い方 – Qiita](https://qiita.com/hana_shin/items/9a88832b4e9e0f082b9c)
[SSHポートフォワーディング（トンネリング）とは: 小粋空間](http://www.koikikukan.com/archives/2016/09/15-000300.php)
[【改訂版】SSHポートフォワード（SSHトンネル）【ローカル・リモート・ダイナミック総集編】 | ITログ](https://www2.filewo.net/wordpress/2019/03/31/【改訂版】sshポートフォワード（sshトンネル）【ロ/)
[sshポートフォワーディングで詰まったら確認すること – Qiita](https://qiita.com/isotai/items/f8ece84816f34eb515e5)
[いますぐ実践! Linuxシステム管理 / Vol.102](http://www.usupi.org/sysad/102.html)

Agentフォワーディングについて
[ssh agent forwardingを行うサーバー側の要点の備忘録 – Qiita](https://qiita.com/hirotaka-tajiri/items/5197c8fa7f32d766c9cc)
[ssh-agentを利用して、安全にSSH認証を行う – Qiita](https://qiita.com/naoki_mochizuki/items/93ee2643a4c6ab0a20f5)
[ssh agent forwarding · JoeMPhilips](http://joemphilips.com/post/ssh_agent_forwarding/)

ゲートウェイポートについて
[異なるprivateネットワーク内の端末をsshで繋ぐ – Qiita](https://qiita.com/FGtatsuro/items/e2767fa041c96a2bae1f)