# HTML更新（リモートにないディレクトリを検知して自動で作成）

## やりたいこと

[前回のスクリプト](sync.html)を改造し、Windowsの特定のフォルダごと`scp`コマンドでアップロードしたとき、ローカルのフォルダ構造に合わせてリモートでも自動でディレクトリを作る。

もう少し具体的に言うと

~~~
(ローカル)
test
　└html
　　├image
　　│　├rsa_key
　　│　│　└image1.png
　　│　└sync2
　　│　　 └image2.png　←同じフォルダ構造でアップロードしたいファイル
　　├html.html
　　└html.md
　　
(リモート)
www
　└html
　　├image
　　│　└rsa_key
　　│　 　└image.png
　　├html.html
　　└html.md
~~~

みたいになっているとき

~~~
scp: www/html/image/sync2/image2.png: No such file or directory
~~~

とか言われてアップロードできない問題を解決する。これは`www/html/image/sync2`というディレクトリが無いのが問題。なのでそれを検知して自動で必要なディレクトリを作るように[前回](sync.html)のスクリプトを改造する。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4

## スクリプト

[前回](sync.html)作ったスクリプト`sync.ps1`を以下に書き換えて、ユーザー名、ラズパイIP、ポート番号は自分で使っているものに書き換えて、保存する。

~~~powershell
$targetFolderName = "\test"
$remoteFolder = "/home/(ユーザー名)/www"
$remoteAddress = "upload@(ラズパイのIP)"
$keyName = "id_rsa"
$datName = "lasttime.dat"
$port = (ポート番号)

#--------------------------------------------------------------

Set-Location $PSScriptRoot

[datetime]$lastdate = (Get-Content $datName)
$targetFolder = $PSScriptRoot + $targetFolderName
$dirlist = (ssh $remoteAddress -i $keyName -p $port "tree -f -i -d --noreport $remoteFolder")

$itemList = Get-ChildItem $targetFolder -Recurse -File
foreach($item in $itemList)
{
    if ($item.LastWriteTime -gt $lastdate)
    {
        $s = (Split-Path $item.FullName -Parent)
        $localFolderName = $s.Substring($targetFolder.Length, $s.Length - $targetFolder.Length).Replace("\", "/")
        if($localFolderName -eq ""){
            $IsNewFolder = $false
        } else {
            $IsNewFolder = $true
            if($dirlist.GetType().Name -ne "String"){
                for ($i = 1; $i -le $dirlist.Length; $i++)
                {
                    $s = $dirlist[$i]
                    if ("$remoteFolder$localFolderName" -eq $s)
                    {
                        $IsNewFolder = $false
                        break
                    }
                }
            }
        }
        if ($IsNewFolder -eq $true)
        {
            ssh $remoteAddress -i $keyName -p $port "mkdir $remoteFolder$localFolderName"
            if($dirlist.GetType().Name -ne "String"){
                $dirlist += "$remoteFolder$localFolderName"
            } else {
                $dirlist = "$dirlist", "$remoteFolder$localFolderName"
            }
        }
        $fileName = $item.FullName.Substring($targetFolder.Length, $item.FullName.Length - $targetFolder.Length)
        scp -P $port -i $keyName $item.FullName "${remoteAddress}:$remoteFolder$fileName"
    }
}

(Get-Date).DateTime > $datName
$a = Read-Host "Press Enter"
~~~

## 解説

### 変更点

* セミコロン`;`を取った。

* 定数を些細なレベルで増やした。

* 以下の2点を追加。

  ~~~powershell
  $dirlist = (ssh $remoteAddress -i $keyName -p $port "tree -f -i -d --noreport $remoteFolder")
  ~~~

  ~~~powershell
  $s = (Split-Path $item.FullName -Parent)
  $localFolderName = $s.Substring($targetFolder.Length, $s.Length - $targetFolder.Length).Replace("\", "/")
  if($localFolderName -eq ""){
      $IsNewFolder = $false
  } else {
      $IsNewFolder = $true
      if($dirlist.GetType().Name -ne "String"){
          for ($i = 1; $i -le $dirlist.Length; $i++)
          {
              $s = $dirlist[$i]
              if ("$remoteFolder$localFolderName" -eq $s)
              {
                  $IsNewFolder = $false
                  break
              }
          }
      }
  }
  if ($IsNewFolder -eq $true)
  {
      ssh $remoteAddress -i $keyName -p $port "mkdir $remoteFolder$localFolderName"
      if($dirlist.GetType().Name -ne "String"){
          $dirlist += "$remoteFolder$localFolderName"
      } else {
          $dirlist = "$dirlist", "$remoteFolder$localFolderName"
      }
  }
  ~~~

### `ssh`コマンドでLinuxコマンドを送る

これまでの`ssh`コマンドの使い方だと、ログインするためのような感じだけど、

~~~shell
> ssh (ユーザー名)@(ラズパイのIP) Linuxコマンド
~~~

という書式で打てば、コマンドだけを実行して返してくれる。（実際はログインしてコマンドを実行した後すぐログアウトする）

なので、

~~~powershell
$dirlist = (ssh $remoteAddress -i $keyName -p $port "tree -f -i -d --noreport $remoteFolder")
~~~

これは

1. `ssh`コマンドでログインし
2. `tree -f -i -d --noreport $remoteFolder`を実行し
3. すぐログアウトをし
4. 2.のコマンドの実行結果を変数`$dirlist`に格納している

ということになる。

### Linuxコマンド：`tree`

ディレクトリの構造をツリー状に表示してくれるコマンド。`-f`はパス付で表示、`-d`でディレクトリのみ表示。`-i`でツリー状にせずに表示。`--noreport`は最後に表示するレポート行（表示したファイル数とディレクトリを表示している行）を表示しないオプション。

もし対象のディレクトリがサブディレクトリを持っていないとき、`$dirlist.GetType().Name`は`String`を返し、サブディレクトリを持てば配列を返す。

参考：[【 tree 】コマンド――ディレクトリをツリー状に表示する：Linux基本コマンドTips（179） \- ＠IT](https://www.atmarkit.co.jp/ait/articles/1802/01/news025.html)

### PowerShellコマンドレット：`Split-Path`

`"C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\CONFIG\machine.config"`みたいなパスを`"C:\WINDOWS\Microsoft.NET\Framework\v2.0.50727\CONFIG"`と`"machine.config"`に割ってくれるようなコマンドレット。

参考：[WindowsのPowerShellでパス文字列を操作する：Tech TIPS \- ＠IT](https://www.atmarkit.co.jp/ait/articles/0809/12/news139.html)