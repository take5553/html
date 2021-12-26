# とにかくPHPとMySQLを動かす

## 前提

LAMPといえば

* `L`・・・Linux
* `A`・・・Apache
* `M`・・・MySQL
* `P`・・・PHP

のことだけど、このうち`L`はOSのことであり、`A, M, P`が動けば何でもいいんじゃね？と勝手に決めつけて特に考慮しないことにする。

さらに`P`のDockerイメージは`A`のApacheとセットになった`php:<version>-apache`というものがあり、「どうせApacheなんてWebサーバーってだけなんだから、丁寧にバージョン合わせる必要無いでしょ」という楽観的視点から細かく考慮しないことにする。

つまり頑張るのは`M`と`P(+A)`の2つ。

もし将来これでエラーが出たら泣きながら頑張る。ちなみにPHP-ApacheコンテナはDebianベースなのでその辺に注意する。

## 参考

[DockerでサクッとLAMP環境を構築してプログラムを動作させたい|やまでぃーのブログ](https://yama-itech.net/construct-lamp-with-docker)

## 手順

### DockerとDocker Composeをインストール

#### Linux(Arch Linux)の場合

ターミナルで以下を打つ。

~~~shell
$ sudo pacman -Syu
$ sudo pacman -S docker
$ sudo pacman -S docker-compose
~~~

#### Windowsの場合

（作成中）

#### インストール確認

~~~shell
$ docker -v
Docker version 20.10.12, build e91ed5707e
~~~

~~~shell
$ docker-compose -v
Docker Compose version 2.2.2
~~~

### 作業ディレクトリを作成

適当なところに適当な名前`docker`のディレクトリ（フォルダ）を作る。中に以下のような構成を目指す。

~~~
docker
├── docker-compose.yml
├── source
|   └── test(空ディレクトリ)
├── mysql
|   └── data(空ディレクトリ)
└── php
    ├── php.ini
    └── Dockerfile
~~~

ホームディレクトリ（もしくはどこか適当なところ）に移動して以下を打つ。

~~~shell
$ mkdir docker
$ cd docker

$ mkdir php
$ touch php/Dockerfile

$ mkdir mysql
$ mkdir mysql/data

$ mkdir source
$ mkdir source/test

$ touch docker-compose.yml
~~~

### `docker-compose.yml`の中身

以下のように編集する。

~~~yaml
services:
  mysql:
    image: mysql:(指定のバージョン)
    volumes:
      - ./mysql/data:/var/lib/mysql
    ports:
      - 3306:3306
    environment:
      - MYSQL_ROOT_PASSWORD=(rootのパスワード何でも)
      - MYSQL_DATABASE=(指定のDB名)
      - MYSQL_USER=(指定のユーザー名)
      - MYSQL_PASSWORD=(指定のユーザーのパスワード)
  php:
    build: ./php
    volumes:
      - ./php/php.ini:/usr/local/etc/php/php.ini
      - ./source/test:/var/www/html
    ports:
      - 80:80
    depends_on:
      - mysql
~~~

### `Dockerfile`の中身

以下のように編集する。

~~~dockerfile
FROM php:(指定のバージョン)-apache
RUN apt-get update && apt-get install -y libonig-dev && \
  docker-php-ext-install pdo_mysql mysqli mbstring
~~~

### `php.ini`の中身

自分の直属の偉い人に「`php.ini`が欲しい」と言えばくれるのではないか。それをコピーしてくる。

### コンテナの立ち上げ

`docker-compose.yml`があるディレクトリで以下を打つ。

~~~shell
$ sudo docker-compose up -d
~~~

立ち上がり確認は、以下を打ち2つのコンテナが見えたらOK。見えなかったら頑張る。

~~~shell
$ sudo docker ps
~~~

## 動作確認

### HTMLのみ

`source/test`ディレクトリに、以下の内容で`index.html`を作成する。

~~~html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World</h1>
</body>
</html> 
~~~

ブラウザから`localhost`にアクセスして大きくHello Worldと表示されたらOK。

### HTML+PHP

`source/test`ディレクトリの`index.html`を`index.php`にリネームして、内容を以下のように変更する。

~~~php+HTML
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World</h1>
    <?php print(date("Y-m-d h:m:s")) ?>
</body>
</html> 
~~~

Hello Worldの下に現在時刻が表示されたらOK。

### HTML+PHP+MySQL

まずMySQL内に適当なデータを作ってあげる必要がある。

ターミナル上で以下を打つとコンテナ内のMySQLにログインできる。

~~~shell
$ sudo docker exec -it (コンテナ名) mysql -u (指定のユーザー名) -p
~~~

コンテナ名は以下で調べることができる。コンテナを表す行の一番右がコンテナ名になっている。自分の場合は`docker-mysql-1`ってなってた。

~~~shell
$ sudo docker ps
~~~

コンテナ内で`mysql`コマンドが実行されるとパスワードを要求されるから、`docker-compose.yml`に書いた指定のユーザーのパスワードを入力してログインする。ログインしたら以下のようなプロンプトが表示されるはず。

~~~mysql
mysql>
~~~

とりあえず`docker-compose.yml`の中に書いた指定のDB名はできているはずなので、それを使う。

~~~mysql
> use (指定のDB名);
~~~

さらに以下を打つ。

~~~shell
> create table test(id int, name varchar(10));
> insert into test values(1, "hogehoge");
~~~

これで`test`テーブルと、その中に1行レコードが作成された。これをPHPで引っ張ってこれたらOK。

とりあえずMySQLから出る。

~~~mysql
> exit
~~~

出てきたら`source/test`ディレクトリ内の`index.php`を以下のように変更する。

~~~php+HTML
<?php
$db["host"] = "mysql";
$db["user"] = "(指定のユーザー名)";
$db["pass"] = "(指定のユーザーのパスワード)";
$db["dbname"] = "(指定のDB名)";
$mysqli = new mysqli("mysql", "(指定のユーザー名)", "(指定のユーザーのパスワード)");
if ($mysqli->connect_errno) {
    print("failed connecting... " . $mysqli->connect_error);
    exit();
}
$mysqli->select_db("(指定のDB名)");
$query = "SELECT * FROM test;";
$result = $mysqli->query($query);
while ($row = $result->fetch_assoc()){
    $content = $row["name"];
}
?>

<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Hello World</title>
</head>
<body>
    <h1>Hello World</h1>
    <?php print(date("Y-m-d h:m:s")) ?><br>
    <?php print($content) ?>
</body>
</html> 
~~~

保存終了して`localhost`にアクセスして、現在時刻の下に`hogehoge`の文字が見えたら勝ち。
