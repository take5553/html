# Gitの設定

## ユーザー名とメールアドレスの設定

~~~powershell
> git config --global user.name "（名前）"
> git config --global user.email （メールアドレス）
~~~

## エディターの設定

~~~powershell
> git config --global core.editor nvim
~~~

## `git log`で文字化けするときの対処

環境変数`LESSCHARSET`に`utf-8`をセット。

~~~powershell
> [System.Environment]::SetEnvironmentVariable("LESSCHARSET", "utf-8", "User")
~~~

