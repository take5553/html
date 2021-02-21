# 事前調査　～基本アイディア～

開発を始める前のメモ的なもの。

## 環境

Python 3

GPIO（Raspberry Pi 3B+）

## 基本アイディア

* 以下をCRONで定期的に実行

  1. PythonでDBにアクセス

     「Python MySQL」でググると、どうも`mysql-connector-python`というライブラリが使えるっぽい。これで記事IDの最大値をするSQL文を実行する。

  2. Pythonでファイルの読みこみ

     保存した記事IDの最大値をファイルから読み出し、DBから取得した記事IDの最大値と比べる。

  3. GPIOディレクトリの操作でLEDを光らせる

     比較結果を元に`/sys/class/gpio/gpio17/value`に`0`または`1`を書き込む

* メインPCからRaspberry Piにアクセスし、記事IDの最大値を更新

  上記の2.のファイルを書きなおすスクリプトを書き、メインPCのデスクトップにショートカットを作っておく。