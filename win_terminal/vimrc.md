# NeoVimの設定（vimrc）

## 準備

~~~powershell
> mkdir .nvim/swp
> mkdir .nvim/tilde
> mkdir .nvim/un
~~~

## 手順

まずNeoVimにとってのvimrcである`C:\Users\（ユーザー名）\AppData\Local\nvim\init.vim`を作成する。

~~~powershell
> mkdir C:\Users\（ユーザー名）\AppData\Local\nvim
> vim C:\Users\（ユーザー名）\AppData\Local\nvim\init.vim
~~~

そのまま以下を記述

~~~
set number

set hlsearch
set ignorecase
set incsearch
set smartcase

set laststatus=2
syntax on
set autoindent
set showcmd

set mouse=a

set directory=~\.nvim\swp
set backupdir=~\.nvim\tilde
set undodir=~\.nvim\un

set expandtab
set tabstop=4
set shiftwidth=4
set softtabstop=4

set encoding=utf-8
set fileencoding=utf-8
set fileencodings=ucs-bom,utf-8,cp932,iso-20220-jp,euc-jisx-0213,euc-jp,guess

"編集箇所のカーソルを記憶
if has("autocmd")
    autocmd bufReadPost *
            \ if line("'\"") > 0 && line("'\"") <= line("$") |
            \   exe "normal! g'\"" |
            \ endif
endif
~~~

