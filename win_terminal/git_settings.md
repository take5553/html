# Gitの設定

## `git log`で文字化けするときの対処

環境変数`LESSCHARSET`に`utf-8`をセット。

~~~powershell
> [System.Environment]::SetEnvironmentVariable("LESSCHARSET", "utf-8", "User")
~~~

