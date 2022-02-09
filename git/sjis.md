# WindowsでDiffが文字化け

## 1. `.gitattributes`を作成し、ファイルごとにエンコードを指定

`.git`ディレクトリがある場所に`.gitattributes`を作成し以下の内容にする。

例えば`.txt`ファイルでエンコードを指定

~~~
*.txt diff=sjis
~~~

## 2. `nkf`コマンドの導入

[nkf.exe nkf32.dll Windows用の詳細情報 : Vector ソフトを探す！](https://www.vector.co.jp/soft/win95/util/se295331.html)

ここから`nkfwin.zip`をダウンロードし、中から出てくる`nkf32.exe`を`nkf.exe`にリネームしてパスの通ったフォルダに入れる。

## 3. Gitのオプション指定

ターミナルで以下を打つ。

~~~shell
> git config diff.sjis.textconv "nkf -w"
~~~

リポジトリごとの設定になるので、グローバルに設定したいときは以下。

~~~shell
> git config --global diff.sjis.textconv "nkf -w"
~~~

この時点でSourceTreeなんかでは再起動すると日本語が読めるようになる。

## 4. ターミナルで`git diff`したいとき

~~~shell
> git config core.pager "LESSCHARSET=utf-8 less"
~~~

または

~~~shell
> git config --global core.pager "LESSCHARSET=utf-8 less"
~~~

