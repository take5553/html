# HTML更新（D&Dでアップロード）

## やりたいこと

* ローカルマシン（PC等）の`D:\`直下に`update1.html`という名前のhtmlファイルがすでに出来上がっている
* Raspberry Pi上のNginxのドキュメントルート（公開するhtmlファイルなどを入れておく場所）が`~/www/html`

この状態のとき、htmlファイルをドラッグアンドドロップしたらそれを`scp`でRaspberry Pi上にアップロードしてくれるようなPowerShellスクリプトを作成する。

## 環境

+ ローカル（PC側）
  * Windows10
  * Powershell 5.1
+ リモート（Raspberry Pi）
  * Raspberry Pi 3B+
  * Raspberry Pi OS 10.4

## 方針

* PowerShellの実行ポリシーの確認＆変更
* PowerShellスクリプト作成

秘密鍵にパスフレーズが設定されており、その入力を回避するなら

* 新たにパスフレーズ無しのRSA鍵ペアを作成
* アップロード用ユーザー`upload`を作成
* ドキュメントルートの権限を変更
* `upload`ユーザーがログインする用の公開鍵を登録
* `upload`ユーザーがドキュメントルートにファイルの追加ができることを確認
* PowerShellスクリプト編集

## 方法

### PowerShellの実行ポリシーの変更

~~~shell
> Set-ExecutionPolicy RemoteSigned
~~~

### PowerShellスクリプトの作成

以下をメモ帳等のテキストエディタに貼り付けて、変えるところを適当に変えて、`upload.ps1`という名前で適当に保存。

~~~powershell
foreach ($file in $args) {
	scp $file (ラズパイのユーザー名)@(ラズパイのIP):~/www/html
}

$a = Read-Host "エンターキーを押してください"
~~~

今作成したps1ファイルのショートカットをデスクトップに作り、右クリックでプロパティを開いてリンク先に書かれている文字列（ファイルパス）の前に

~~~
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File <元の文字列>
~~~

と追記する。

ショートカットにファイル（複数可）をドラッグアンドドロップすると

~~~
Enter passphrase for key '(秘密鍵のパス)':
update1.html                                   100%   55KB   2.8MB/s   00:00
エンターキーを押してください:
~~~

アップロードに成功する。これでわざわざコマンドを打たなくても`scp`コマンドでファイルのアップロードができるようになった。

ただ、秘密鍵にパスフレーズが設定されている場合、その入力は毎回必要になる。パスフレーズはセキュリティのことを考えると設定する方がいいけど、今回のように利便性は少し失われる。

パスフレーズ入力を回避するためにアップロード専用ユーザー`upload`を作成し、HTMLファイル更新のための最低限の権限を与えるようにする。

### パスフレーズ無しのRSA鍵ペアを作成

ローカルで作業する。パスフレーズを設定しないRSA鍵（新規ユーザー用）の作成。

~~~shell
> ssh-keygen
~~~

コマンドを打つと以下の入力を求められる。ここでは適当なフォルダの下に`id_rsa`というファイル名で出力することにする。

~~~shell
Enter file in which to save the key (C:\Users\****/.ssh/id_rsa): (保存ファイル名をフルパスで入力)
Enter passphrase (empty for no passphrase):(未入力のままエンター)
Enter same passphrase again:(未入力のままエンター)
~~~

入力が終了すると以下のように表示される。

~~~
Your identification has been saved in (先ほど指定したファイル名).
Your public key has been saved in (先ほど指定したファイル名).pub.
The key fingerprint is:
SHA256:(ハッシュ化された文字列)
The key's randomart image is:
+---[RSA 2048]----+
|　鍵固有のイメージ  |
~~~

これで指定フォルダの下に`id_rsa`という拡張子無しのファイルと`id_rsa.pub`というファイルが生成される。

### アップロード用ユーザー`upload`を作成

リモートで作業する。アップロード用の新規ユーザー`upload`の作成と念のためパスワードの設定

~~~shell
$ sudo useradd upload -m
$ sudo passwd upload
~~~

ユーザーを作ったら、同時に同名のグループもできる。名前もちょうどいいので`upload`グループを使っていくことにする。一応既存ユーザーを`upload`グループに登録しておく。

~~~shell
$ sudo usermod -aG upload (既存ユーザー名)
~~~

### ドキュメントルートの権限を変更

ドキュメントルートを`upload`グループに所属させ、書き込み権限を与える。

~~~shell
$ chown -R (既存ユーザー名):upload ~/www
$ chmod -R 775 ~/www
~~~

### `upload`ユーザーがログインする用の公開鍵を登録

リモートからログアウトしてローカルで作業。

~~~shell
$ exit
~~~

一旦既存ユーザーのホームディレクトリに公開鍵をアップロード。

~~~shell
> scp (公開鍵の場所のフォルダパス)\id_rsa.pub (既存ユーザー)@(ラズパイのIP):~/authorized_keys
~~~

Raspberry Piに既存ユーザーでログインし、`upload`ユーザーのホームディレクトリに公開鍵を移す。

~~~shell
$ sudo mkdir /home/upload/.ssh
$ sudo mv authorized_keys /home/upload/.ssh
~~~

所有者＆権限の設定。

~~~shell
$ cd /home/upload
$ sudo chown -R upload:upload .ssh
$ sudo chmod 700 .ssh
$ sudo chmod 600 .ssh/authorized_keys
~~~

念のために、既存ユーザーをログアウト。`upload`ユーザーでログインできることを確認。

~~~shell
$ exit
(ローカルから)
$ ssh upload@(ラズパイのIP) -i (uploadユーザー用の秘密鍵の場所のパス)\id_rsa
~~~

`upload`ユーザーでログインできることを確認。

~~~shell
> ssh upload@(ラズパイのIP) -i (uploadユーザー用の秘密鍵の場所のパス)\id_rsa
~~~

### `upload`ユーザーがドキュメントルートにファイルの追加ができることを確認

Raspberry Piに`upload`ユーザーでログインする。

~~~shell
$ cd /home/(既存ユーザー名)/www/html
$ touch a.html
$ ls
a.html  index.html  update1.html
$ rm a.html
index.html  update1.html
~~~

### PowerShellスクリプト編集

`upload.ps1`ファイルを以下のように書き換える。

~~~powershell
foreach ($file in $args) {
	scp $file upload@(ラズパイのIP):/home/(既存ユーザー名)/www/html -i (uploadユーザー用の秘密鍵の場所のパス)\id_rsa
}

$a = Read-Host "エンターキーを押してください"
~~~

何か適当にD&D。

~~~
update1.html                               100%   83KB   3.3MB/s   00:00
エンターキーを押してください:
~~~

成功。

## 解説

### PowerShellスクリプトを学ぶ

[PowerShell 使い方メモ \- Qiita](https://qiita.com/opengl-8080/items/bb0f5e4f1c7ce045cc57)

本を買わなくてもいいレベルで詳しい。

### `chown`コマンド

ファイルやディレクトリの所有者＆所属グループを変更するコマンド。記事中では

~~~shell
$ chown -R (既存ユーザー名):upload ~/www
~~~

や

~~~shell
$ sudo chown -R upload:upload .ssh
~~~

という形で紹介した。

書式は

~~~shell
$ chown (所有者):(所属グループ) 対象となるファイルまたはディレクトリ
~~~

で、他ユーザーの所有物をいじるときは`sudo`が必要。

* ディレクトリの中身も再帰的に変更するオプション：`-R`

  これを付けないとディレクトリだけ所有者やグループを変更し、その中身はそのままになる。中身も含めて丸ごと変更したい場合はこのオプションを付ける。

* 実行状況を表示させるオプション：`-v`

  変更したのかしてないのか普通は表示しないが、このオプションを付けると何をどう変更したのか表示させることができる。

### `chmod`コマンド

書式は

~~~shell
$ chmod (権限の状態を表す3桁の数字) 対象となるファイルまたはディレクトリ
~~~

* 3桁の数字の説明

  ~~~shell
  $ chmod -R 775 ~/www
  ~~~

  これは「`www`ディレクトリの権限を`rwxrwxr-x`に変更しなさい」というコマンド（`-R`は`chown`と同じで、ディレクトリの中身も一緒に再帰的に変更するオプション）。[前回記事](update1.html)で権限の読み方を解説したが、

  * `r=4`
  * `w=2`
  * `x=1`

  として、その合計で`r`、`w`、`x`の組み合わせを1桁の数字で表現する。組み合わせとして`0`から`7`まであり

  * `0:---`
  * `1:--x`
  * `2:-w-`
  * `3:-wx`
  * `4:r--`
  * `5:r-x`
  * `6:rw-`
  * `7:rwx`

  となる。

  数字が3桁あるのは

  * 1桁目：所有者に対する権限
  * 2桁目：所属グループのメンバーに対する権限
  * 3桁目：それ以外のユーザーに対する権限

  を表している。

  なので、公開鍵登録のところで出てきた

  ~~~shell
  $ sudo chmod 700 .ssh
  $ sudo chmod 600 .ssh/authorized_keys
  ~~~

  は、それぞれ

  * `.ssh`ディレクトリは`rwx------`
  * `.ssh/authorized_keys`は`rw-------`

  という権限を設定している。所有者にしか権限を設定していないことになる。

