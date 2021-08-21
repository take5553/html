# コンテナ上でGUIアプリ開発　環境構築2

どこでコードを書くか。

## Linuxから（メインPC）

シンプルにこんな感じ。

![image-20210809175731520](image/gui_development2/image-20210809175731520.png)

`scp`コマンドで飛ばして、`ssh`でコマンド実行。なんか泥臭いけど、困るまではこれで行く。

前提として

* Jetson Nanoへのログインをパスフレーズ無しの公開鍵で行う
* Jetson Nanoユーザーが`docker`コマンドを打つときに`sudo`をつけなくてもいいようにしておく
* すでにJetson Nanoへのログインをしていて、すでにコンテナが動いている

ことが必要。GUIをローカルに飛ばしてくるのは、ここに来るまでにすでにやっているので割愛。

### コマンド作成

`scp`と`docker exec`を1コマンドで行えるようなコマンドを作ってしまえば良い。

適当にテキストファイルに以下を記入。

~~~
scp ~/python/pytorch_test/test2.py takeshi@192.168.1.205:~/my-docker/pytorch/src
ssh takeshi@192.168.1.205 docker exec pytorch_test_1 python3 /pytorch/src/test2.py
~~~

実行権限を与えて、パスが通っているディレクトリに放り込めば完了。

ファイルが増えてきたりしたら、また対策を考えよう。

### 実機と併用するとき

今まではJetson NanoはあくまでもSSH接続のみとしていたけど、開発が進んだらやはり実機でもデバッグしたいということで、実機で立ち上げつつローカルPCでもデバッグするときの対処をメモしておく。

ポイントは環境変数`$DISPLAY`の値で、

* 実機でデスクトップが立ち上がっている状態→`:0`
* SSH接続＋`-Y`オプション→`localhost:10.0`

となっている。

`docker-compose up -d`でコンテナを立ち上げるときに`$DISPLAY`の内容も引き継いでいるので、どっちでコンテナを立ち上げるかによってセットされる環境変数が違う。

なので、ローカルから遠隔でテストするときは強制的に`$DISPLAY`の内容を`localhost:10.0`にしてやれば良い。つまり以下。

~~~
scp ~/python/pytorch_test/test2.py takeshi@192.168.1.205:~/my-docker/pytorch/src
ssh takeshi@192.168.1.205 docker exec -e DISPLAY=localhost:10.0 pytorch_test_1 python3 /pytorch/src/test2.py
~~~

`docker exec`の後ろの`-e DISPLAY=localhost:10.0`を加えただけ。

## Windowsから（ノートPC）

基本的な構造は同じ。ただ、`scp`コマンドの代わりにWindowsからはVSCodeのRemote Developmentが使える。でも、GUIを手元に飛ばしてくる方法を準備してあげないといけない。

（準備中）
