# HTML更新（ダブルクリックでフォルダごとアップロード）

## やりたいこと

Powershellスクリプトで最終更新日以降のファイルのみを`scp`コマンドでRaspberry Piに送信。デスクトップにPowerShellスクリプトのショートカットを作っておき、それをダブルクリックすると実行できるようにする。

## 環境

+ ローカル（PC側）
  * Windows10
  * Powershell 5.1
+ リモート（Raspberry Pi）
  * Raspberry Pi 3B+
  * Raspberry Pi OS 10.4

## Powershell スクリプト

以下のスクリプトをメモ帳に貼り付けて、必要なところを適宜変更して、`sync.ps1`という名前で保存。

~~~powershell
$targetFolderName = "\test";
$remoteFolder = "upload@(ラズパイのIP):/home/(ユーザー名)/www";
$keyName = "\rsa_id";
$datName = "lasttime.dat";

#--------------------------------------------------------------

Set-Location $PSScriptRoot;

[datetime]$lastdate = (Get-Content $datName);
$targetFolder = $PSScriptRoot + $targetFolderName;

$itemList = Get-ChildItem $targetFolder -Recurse -File;
foreach($item in $itemList)
{
    if ($item.LastWriteTime -gt $lastdate)
    {
        $fileName = $item.FullName.Substring($targetFolder.Length, $item.FullName.Length - $targetFolder.Length);
        scp -i $PSScriptRoot$keyName $item.FullName $remoteFolder$fileName;
    }
}

(Get-Date).DateTime > $datName;
$a = Read-Host "Press Enter";
~~~

保存場所

~~~
任意のフォルダ
　├id_rsa
　├id_rsa.pub
　├lasttime.dat　※自動生成
　├sync.ps1
　└test　※アップロードしたいフォルダ
　　└色々なファイル
~~~

`sync.ps1`と同じフォルダに`upload`ユーザー用の秘密鍵と、アップロードしたいフォルダを置く必要がある。

`lasttime.dat`はプログラムが自動で生成するファイルで、最終更新日が書き込まれている。

## 解説

~~~powershell
$targetFolderName = "\test";
$remoteFolder = "upload@(ラズパイのIP):/home/(ユーザー名)/www";
$keyName = "\rsa_id";
$datName = "lasttime.dat";

#--------------------------------------------------------------
~~~

定数たち。ラズパイのIPとユーザー名は各自の環境に書き換えてください。

~~~powershell
Set-Location $PSScriptRoot
~~~

`$PSScriptRoot`の中にPowerShellスクリプトが置かれているフォルダのフルパスが入っているので、そこに移動する。

※ということはスクリプト起動時のカレントフォルダは別の場所にあるということ。PowerShellスクリプトと同じフォルダ内にあるファイルを操作したければこのテクニックは必須。

~~~powershell
[datetime]$lastdate = (Get-Content $datName);
$targetFolder = $PSScriptRoot + $targetFolderName;
~~~

処理に必要な値の取得。`$datName`の中に前回の更新時刻が入っている。

~~~powershell
$itemList = Get-ChildItem $targetFolder -Recurse -File;
foreach($item in $itemList)
{
	略
}
~~~

同期させるフォルダの中身をサブフォルダも含めて全部取ってきて、１つ１つに対して処理を行う。

~~~powershell
    if ($item.LastWriteTime -gt $lastdate)
    {
        $fileName = $item.FullName.Substring($targetFolder.Length, $item.FullName.Length - $targetFolder.Length);
        scp -i $PSScriptRoot$keyName $item.FullName $remoteFolder$fileName;
    }
~~~

ファイルが前回の更新時刻以降に更新されていたら、ファイル名（サブフォルダ内にあればサブフォルダ名＋ファイル名）を取得して、`scp`コマンドでRaspberry Piに送信。

Raspberry PiのSSH接続ポートを変えているなら、この`scp`コマンドに`-P （ポート番号）`のオプションを付けること。

~~~powershell
(Get-Date).DateTime > $datName;
~~~

現在時刻を前回更新時刻としてファイルに保存。

~~~powershell
$a = Read-Host "Press Enter";
~~~

処理後すぐPowerShellのウィンドウが消えるのを防ぐ。

## 欠点

* ただ`scp`でRaspberry Piに送り付けるだけなので、Raspberry Pi上のファイルを削除したりはしない。→完全な同期ではない。
* PC1台とRaspberry Piのみで考えており、例えば自宅ではPCで、出先ではノートパソコンから更新しようと思ったらこのスクリプトじゃなくてGitを使った方がいいかも。