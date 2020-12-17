# GitHubからRaspberry PiににPullする処理を自動化

ダブルクリックでRaspberry PiがGitHubから自動でPullするようなスクリプトショートカットをデスクトップに作成する。

## 環境

- ローカル
  - Windows 10
  - git version 2.28.0.windows.1
- リモート
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - git version 2.20.1

## 概要

ほぼ[以前にやったこと](../webserver/syncgit.html)の流用。SSHコマンドで

* Raspberry Pi上のワークフォルダに移動せよ
* `git pull origin master`を実行せよ

を送信する。

## PowerShellスクリプト

### PowerShellの実行ポリシーの変更

PowerShellで以下を打つ。

```shell
> Set-ExecutionPolicy RemoteSigned
```

### PowerShellスクリプトの作成

`php-bbs-pull.ps1`という名前で以下の内容を保存。

~~~powershell
$remote = "upload@(Raspberry PiのIP)"
$privateKey = "(uploadユーザーの秘密鍵への(ローカル上での)パス)"
$port = "(22または変更したRaspberry Piのポート番号)"
$command = 'cd /home/(ユーザー名)/www/html/php-bbs && git pull origin master'

ssh -i $privateKey -p $port $remote $command
$a = Read-Host "Press Enter"
~~~

`php-bbs-pull.ps1`のショートカットをデスクトップに作成し、右クリックでプロパティを開いて、「リンク先」に書いてある文字列の前に以下を付け足す。

```
C:\Windows\System32\WindowsPowerShell\v1.0\powershell.exe -File (元の文字列)
```

## 実験

VSCode上で何かを変え保存しコミット＆プッシュ。

内容

~~~php
<?php
echo 'hogehoge';
~~~

そして作ったショートカットをダブルクリック。コマンドプロンプトが立ち上がり、SSH経由で`git pull origin master`をし、Enterを押せばコマンドプロンプトが閉じる。

~~~
From github.com:take5553/php-bbs
 * branch            master     -> FETCH_HEAD
   b4ee60b..5bee76a  master     -> origin/master
Updating b4ee60b..5bee76a
Fast-forward
 index.php | 1 +
 1 file changed, 1 insertion(+)
Press Enter:
~~~

確認のため`(Raspberry PiのIPまたはURL)/php-bbs`にアクセス。

~~~
hogehoge
~~~

OK。

## これから

ワークフォルダの中身を編集してコミット＆GitHubにプッシュして、このスクリプトのショートカットをダブルクリックすれば自動でRaspberry Pi上にPullされることとなった。

また、masterブランチにあるものは常に本番環境にPullされるという決まりができて、ブランチの役割が少し明確になった。まだmasterしかないけど。