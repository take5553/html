# Jupyter Notebookをサーバー上のバックグラウンドで動かす

## 環境

RHEL 8

Python 3.9.5

## 手順

### 起動時

~~~shell
$ nohup jupyter notebook >> jupyter.log 2>&1 &
[1] 36116
~~~

* `[1]`・・・ジョブ番号
* `36116`・・・PID

### 確認

~~~shell
$ jobs -l
[1]+ 36116 実行中               nohup jupyter notebook >> jupyter.log 2>&1 &
~~~

`-l`を省略するとPIDが表示されない。

### 終了時

~~~shell
$ kill -s 2 36116
~~~

`-s (シグナル番号)`について

* `2`・・・`SIGINT`（キーボードからの割り込み　`ctrl + C`と同じ）
* `9`・・・`SIGKILL`（強制終了）
* `15`・・・`SIGTERM`（通常の終了指令）

## 解説

### バックグラウンド実行について

~~~shell
$ nohup jupyter notebook >> jupyter.log 2>&1 &
~~~

書式は以下。

~~~shell
$ nohup (コマンド) &
~~~

`nohup`と`&`で挟むことで`(コマンド)`をバックグラウンドで実行し、かつSSH接続を切っても継続するようにできる。

* 末尾の`&`・・・コマンドをバックグラウンドで実行
* `nohup`・・・`SIGHUP`（ハングアップシグナル）を無視した状態でプロセスを起動→SSH接続を切っても処理続行。

### リダイレクト（`>>`や`2>`について）

標準出力の内容をファイルに書き出したいときに使う。

書式としては

~~~shell
$ (標準出力や標準エラー出力があるコマンド) (リダイレクト記号) (出力先ファイル)
~~~

リダイレクト記号は以下

* `>`・・・標準出力の内容をファイルに書き込む（上書き）
* `>>`・・・標準出力の内容をファイルに書き込む（追記）
* `2>`・・・標準エラー出力の内容をファイルに書き込む（上書き）
* `2>>`・・・標準エラー出力の内容をファイルに書き込む（追記）
* `2>&1`・・・標準エラー出力を標準出力と同一にする

ということで以下のコマンドは「標準出力と標準エラー出力の両方とも`jupyter.log`に追記していく」という意味になる。

~~~shell
$ jupyter notebook >> jupyter.log 2>&1
~~~

ちなみに標準エラー出力の内容が必要なかったら、ゴミ箱である`/dev/null`にリダイレクトすればよい。

~~~shell
$ jupyter notebook >> jupyter.log 2> /dev/null
~~~

## 参考

### バックグラウンドとシグナル

[Jupyter Notebookをより便利に使うために、色々まとめ - Qiita](https://qiita.com/ishizakiiii/items/b98bbf8997f039f40058)
[Linuxコマンド(Bash)でバックグラウンド実行する方法のまとめメモ - Qiita](https://qiita.com/inosy22/items/341cfc589494b8211844)
[HUPシグナルとnohupとdisownとバック/フォアグラウンドジョブの理解 - Qiita](https://qiita.com/yushin/items/732043ee23281f19f983)
[Linuxの「シグナル」って何だろう？：“応用力”をつけるためのLinux再入門（16）（1/2 ページ） - ＠IT](https://www.atmarkit.co.jp/ait/articles/1708/04/news015.html)
[Linuxの「ジョブコントロール」をマスターしよう：“応用力”をつけるためのLinux再入門（15）（1/2 ページ） - ＠IT](https://www.atmarkit.co.jp/ait/articles/1707/21/news001.html)
[Linux - プロセスの生成、監視、終了（フォアグラウンドとバックグラウンド）](https://www.infraeye.com/study/linuxz17.html)
[プロセスを終了するkillコマンドの使い方まとめ！【Linuxコマンド集】](https://eng-entrance.com/linux-command-kill)

### リダイレクト

[５分で一通り理解できる！Linuxのリダイレクト 使い方と種類まとめ](https://eng-entrance.com/linux-redirect)
