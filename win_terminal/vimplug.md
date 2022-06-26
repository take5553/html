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

~~~
Plug 'tpope/vim-fugitive'
~~~



### vim-airline

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

