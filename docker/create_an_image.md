# Dockerイメージの作り方

もし手元になにかのアプリのソースコードがあったとする。このアプリを動かすための環境を用意するためにイメージファイルをビルドし、コンテナを作成するところを体験する。

## サンプルアプリ

[getting-started/app at master · docker/getting-started](https://github.com/docker/getting-started/tree/master/app)

JS（React）で作られていて配布できる状態。ただし動かすにはnode.jsとpythonが必要で、JSの依存パッケージのインストールも当然必要、という状態。

## Dockerイメージを作成

上記GitHubを`clone`したとして、`app`ディレクトリの中に入り、`Dockerfile`というファイル（拡張子無し）を作成、以下を記述。

~~~dockerfile
 FROM node:12-alpine
 RUN apk add --no-cache python g++ make
 WORKDIR /app
 COPY . .
 RUN yarn install --production
 CMD ["node", "src/index.js"]
~~~

* `FROM node:12-alpine`

  ベースとなるイメージ。node.jsのver12で、OSはAlpine Linuxという軽量のLinux。もしこれが`node:12`だったらOSはDebianになるらしい。

* `RUN apk add --no-cache python g++ make`

  Alpine Linuxのパッケージマネージャーは`apk`らしい。Debianだったらここが`apt`になる。

* `WORKDIR /app`

  コンテナ内のカレントディレクトリを`/app`にする。

* `COPY . .`

  ローカルにあるファイルをコンテナ内にコピーする。`COPY (ソースファイル) (宛先)`という書式。

* `RUN yarn install --production`

  `yarn`コマンドを実行してJSの依存パッケージをダウンロードしている。

* `CMD ["node", "src/index.js"]`

  `RUN`と`CMD`の違いがわかりにくいけど、役割としては同じものらしい。ただ`RUN`はイメージをビルドするときにだけコンテナ内で実行され、`CMD`はイメージからコンテナを作成するときに毎回実行されるらしい。

  つまり、インストールする系のコマンドは`RUN`で、毎回実行して欲しい系は`CMD`ということになる。

## イメージをビルド

`Dockerfile`があるディレクトリで以下を打つ。

~~~shell
$ sudo docker build -t getting-started .
~~~

最後の`.`で、「カレントディレクトリから`Dockerfile`を探してイメージをビルドせよ」というメッセージになるらしい。`-t`はイメージ名を付与するオプション。

これでイメージをビルドすることができる。

## イメージからコンテナを生成

[前回記事](getting_started.html)と同様のことをする。

~~~shell
$ sudo docker run -d -p 3000:3000 --network host getting-started
~~~

コンテナ・イメージの破棄も同様で良い。ただし次以降このサンプルアプリに変更を加えていくので破棄じゃなくてコンテナを止めるだけで良い。

