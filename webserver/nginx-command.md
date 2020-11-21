# Nginxを操作するコマンド

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1

## コマンド

### 立ち上げ

~~~shell
$ sudo nginx
~~~

これで立ち上がる。

また、[Nginxをインストールしたとき](nginx.html)は以下のコマンドで立ち上げた。

~~~shell
$ sudo /etc/init.d/nginx start
~~~

これはサービス（またはデーモン）としての起動という意味で、バックグラウンドで動いているということ。多分。

ちなみにちゃんとサービスとして動いているかどうかの確認は

~~~shell
$ service --status-all
~~~

を打つとサービスがもりもり表示されるので、その中に

~~~
[ + ]  nginx
~~~

があればOK。

または

~~~shell
$ sudo systemctl list-unit-files -t service
~~~

でも確認できる。こっちは大量に表示されるので、矢印キーの↑、↓で表示を移動させ、`ctrl + C`で抜ける。

### 停止

2種類ある。

~~~shell
$ sudo nginx -s stop  # 即停止
$ sudo nginx -s quit  # 応答中のリクエストがあれば、それが終わってから終了
~~~

これは

> このコマンドはnginxを開始したのと同じユーザで実行されるべきです。
>
> [ビギナーのガイド 日本語訳](http://mogile.web.fc2.com/nginx/beginners_guide.html)

だそうだ。大体は`root`ユーザーなので、`sudo`コマンドで大丈夫。

一応確認としては

~~~shell
$ ps aux | grep nginx
~~~

と打てば色々表示されるので、その中の`nginx: master process`と書かれている行の一番左に実行ユーザーが書かれているはず。

### リロード

~~~shell
$ sudo nginx -s reload
~~~

設定ファイルを変更したらこれを打って設定ファイルを読み直す必要がある。一応設定ファイルの妥当性をチェックしてから読み込むらしい。

### ログファイルを開きなおす

~~~shell
$ sudo nginx -s reopen
~~~

用途不明。

### 設定ファイルのテスト

~~~shell
$ sudo nginx -t
~~~

設定ファイルに不備がないかどうかチェックしてくれる。

## 参考

[ビギナーのガイド 日本語訳](http://mogile.web.fc2.com/nginx/beginners_guide.html)
[Nginxインストール編① 無料クラウドAWSでサーバ構築手順まとめ \| TECH Projin \| ページ 2](https://tech.pjin.jp/blog/2014/08/18/nginx%e3%82%a4%e3%83%b3%e3%82%b9%e3%83%88%e3%83%bc%e3%83%ab%e7%b7%a8%e2%91%a0-%e7%84%a1%e6%96%99%e3%82%af%e3%83%a9%e3%82%a6%e3%83%89aws%e3%81%a7%e3%82%b5%e3%83%bc%e3%83%90%e6%a7%8b%e7%af%89%e6%89%8b/2/)