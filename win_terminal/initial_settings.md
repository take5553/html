 # 初期セットアップ

## 参考

[(585) How to set up PowerShell prompt with Oh My Posh on Windows 11 - YouTube](https://www.youtube.com/watch?v=5-aK2_WwrmM)

## Nerd Fontsをインストール

[ryanoasis/nerd-fonts: Iconic font aggregator, collection, & patcher. 3,600+ icons, 50+ patched fonts: Hack, Source Code Pro, more. Glyph collections: Font Awesome, Material Design Icons, Octicons, & more](https://github.com/ryanoasis/nerd-fonts)

Releasesから最新のフォントをダウンロードして、zipファイルを展開して右クリックでインストール。Windows Compatibleをインストールすること。

## Windows Terminalの設定

* デフォルトターミナルアプリケーションをWindows Terminalに変更

  設定の「スタートアップ」にある

* 外観をアクリルに設定

* 既定値の外観

  * カラースキームを「One Half Dark」へ変更
  * Font faceをNerd FontsでDLしてきたものに変更
  * カーソルの形を任意に変更
  * アクリルをON

## PowerShellをインストール

Microsoft Storeから「PowerShell」を検索してインストールする。Windows付属の「Windows PowerShell」と今からインストールする「PowerShell」の違いは、Windows PowerShellは.NET Framework v4.5上で動き、PowerShellは.NET Core 2.0上で動くとのこと。言語的な違いはほとんどないけど、PowerShellはクロスプラットフォームなのでWindowsかそうでないかで動作が変わるようなところだけが違うとのこと。

参考：[Windows PowerShell 5.1 と PowerShell 7.x の相違点 - PowerShell | Microsoft Docs](https://docs.microsoft.com/ja-jp/powershell/scripting/whats-new/differences-from-windows-powershell?view=powershell-7.2)

## Windows Terminalの規定プロファイルを変更

設定の「スタートアップ」にある「規定プロファイル」をPowerShellに変更

## Windows Terminalの背景色を変更

設定画面の左のアイコンの一番したの歯車マークをクリックすると、`settings.json`が直接編集できる。「One Half Dark」で検索してそれっぽいプロファイルを見つけたら、それをまるごとコピーして名前を「One Half Dark(modded)」にする。

その後、`background`という項目を`#001B26`に編集して保存。

設定の「既定値」から外観を選択して、カラースキームを「One Half Dark (modded)」を選択。

## Scoopをインストール

~~~powershell
> iwr -useb get.scoop.sh | iex
~~~

## Scoopを使ってツールをインストール

~~~powershell
> scoop install curl sudo jq
~~~

インストール済みのツールを確認するときは`scoop list`

## Gitのインストール

~~~powershell
> winget install -e --id Git.Git
~~~

ダウンロード後は通常のインストーラーが起動する。

## Neovimをインストール

~~~powershell
> scoop install neovim gcc
~~~

## ユーザープロファイルとコマンドエイリアスを作成

~~~powershell
> mkdir .config/powershell
> nvim .config/powershell/user_profile.ps1
~~~

以下を記述。

~~~powershell
# Alias
Set-Alias vim nvim
Set-Alias ll ls
Set-Alias g git
Set-Alias grep findstr
Set-Alias tig 'C:\Program Files\Git\usr\bin\tig.exe'
Set-Alias less 'C:\Program Files\Git\usr\bin\less.exe'
~~~

そして以下を打つ。

~~~powershell
> nvim $PROFILE.CurrentUserCurrentHost
~~~

以下を記述。

~~~powershell
. $env:USERPROFILE\.config\powershell\user_profile.ps1
~~~

ターミナルを再起動すればエイリアスが有効化される。

## On My Poshをインストール

~~~powershell
> winget install JanDeDobbeleer.OhMyPosh -s winget
~~~

毎回起動するように書き加える。

~~~powershell
> vim .config/powershell/user_profile.ps1
~~~

~~~powershell
# Prompt
oh-my-posh init pwsh | iex
oh-my-posh init pwsh --config ~/.space.omp.json | iex
~~~

## Terminal Iconsをインストール

~~~powershell
> Install-Module -Name Terminal-Icons -Repository PSGallery -Force
~~~

## Zをインストール

~~~powershell
> Install-Module -Name z -Force
~~~

`cd`コマンドで使用するディレクトリを覚えさせること。

## PSReadLineをインストール

~~~powershell
> Install-Module -Name PSReadLine -Scope CurrentUser -Force
~~~

## Fzfをインストール

~~~powershell
> scoop install fzf
> Install-Module -Name PSFzf -Scope CurrentUser -Force
~~~

## ユーザープロファイルを更新

以下を追記

~~~powershell
# Icons
Import-Module -Name Terminal-Icons

# PSReadLine
Set-PSReadLineOption -EditMode Vi
Set-PSReadLineOption -BellStyle None
Set-PSReadLineKeyHandler -Chord 'Ctrl+d' -Function DeleteChar
Set-PSReadLineOption -PredictionSource History

# Fzf
Import-Module PSFzf
Set-PsFzfOption -PSReadlineChordProvider 'Ctrl+f' -PSReadlineChordReverseHistory 'Ctrl+r'
~~~

## `which`コマンドを追加

ユーザープロファイルに以下を追記。

~~~powershell
# Utilities
function which ($command) {
  Get-Command -Name $command -ErrorAction SilentlyContinue |
    Select-Object -ExpandProperty Path -ErrorAction SilentlyContinue
}
~~~

