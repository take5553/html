# SSHログインするためのPowershellエイリアスを設定

## やりたいこと

PowerShellで毎回`ssh`コマンドを打つのが面倒なので、ショートカット的なものを作っておく。

## 環境

- Windows10
- PowerShell 5.1

## 方法

PowerShellを使うと、マイドキュメント内に「WindowsPowerShell」というフォルダができる。その中に`Microsoft.PowerShell_profile.ps1`という名前のテキストファイルを作成し、以下のように書いて保存をする。

~~~powershell
function ssh-takeshi {
	ssh takeshi@192.168.1.201
}
~~~

そうするとPowerShellを立ち上げたときに

~~~shell
> ssh-takeshi
~~~

と打つだけでログインができる。

## 解説

書式は以下。

~~~
function (好きな名前) {
	(PowerShellのコマンド：複数行可)
}
~~~

別に`ssh`コマンドに限らず、PowerShellのコマンド（レット）なら何でも書ける。毎回打つのが面倒な処理を一発で実行することができる。