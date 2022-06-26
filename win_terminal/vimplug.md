# Vimのプラグイン

## vim-plugのインストール

参考：[junegunn/vim-plug: Minimalist Vim Plugin Manager](https://github.com/junegunn/vim-plug)

~~~powershell
> iwr -useb https://raw.githubusercontent.com/junegunn/vim-plug/master/plug.vim |`
    ni "$(@($env:XDG_DATA_HOME, $env:LOCALAPPDATA)[$null -eq $env:XDG_DATA_HOME])/nvim-data/site/autoload/plug.vim" -Force
~~~

## vimrcに追記

~~~
call plug#begin()

Plug '(プラグイン名)'

call plug#end()
~~~

保存したらVimのコマンドラインに以下を打ち込む。

~~~
:PlugInstall
~~~

以下、順番に導入していく。

## 導入プラグイン

### vim-fugitive

[tpope/vim-fugitive: fugitive.vim: A Git wrapper so awesome, it should be illegal](https://github.com/tpope/vim-fugitive)

Vim上でGitが使えるようになる

~~~
Plug 'tpope/vim-fugitive'
~~~

便利そうなのは`:Gvdiff`。色が見にくければWindows Terminalのsettings.jsonで色を調整すること。

### vim-fugitive -blame-ext

[tommcdo/vim-fugitive-blame-ext: Extend tpope/vim-fugitive to show commit message on statusline in :Gblame](https://github.com/tommcdo/vim-fugitive-blame-ext)

上記vim-fugitiveで`:Git blame`した時の画面にコミットメッセージを表示する

~~~
Plug 'tommcdo/vim-fugitive-blame-ext'
~~~

### vim-airline

[vim-airline/vim-airline: lean & mean status/tabline for vim that's light as air](https://github.com/vim-airline/vim-airline)

見た目をよろしくするプラグイン。

~~~
Plug 'vim-airline/vim-airline'
Plug 'vim-airline/vim-airline-themes'
~~~

設定

~~~
"バッファをタブの様に表示
let g:airline#extensions#tabline#enabled = 1
nmap <C-p> <Plug>AirlineSelectPrevTab
nmap <C-n> <Plug>AirlineSelectNextTab
"PowerLineフォントを使用してテーマの変更
let g:airline_powerline_fonts = 1
let g:airline_theme = 'dark'
let g:airline#extensions#whitespace#enabled = 0
~~~

### ack.vim

[mileszs/ack.vim: Vim plugin for the Perl module / CLI script 'ack'](https://github.com/mileszs/ack.vim)

高速な複数ファイル検索。

まず`ack`コマンドをインストール。

~~~powershell
> scoop install ack
~~~

次にプラグインを追加

~~~
Plug 'mileszs/ack.vim'
~~~

