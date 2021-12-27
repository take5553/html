# DBをコピーする

環境やライブラリが揃っても、肝心のDBデータが無いとテストできないので、ここではDBの丸ごとコピーを行う。

## 元DBのDumpファイルを入手する

直属の偉い人に「DBのDumpファイルくれ」と言ったらくれるのではないか。

もしくは自分でDBにアクセスできるのであれば、MySQLにログインできる状態（テストサーバーにSSH接続しているときなど）で以下を打つ。

~~~shell
$ mysqldump -u (ユーザー名) -p (データベース名) > (データベース名).dump.sql
~~~

これをデータベースごとに繰り返す。今は仮に`db1`、`db2`とする。

~~~shell
$ mysqldump -u (ユーザー名) -p db1 > db1.dump.sql
$ mysqldump -u (ユーザー名) -p db2 > db2.dump.sql
~~~

こんな感じ。

おそらくファイルサイズが大きいことが予想されるので、必要に応じて圧縮などして自分の環境に移しておく。自分は`mysql/init_data`というディレクトリを作ってその中に入れた。

## 動作確認で使用したDB内のデータをきれいにしておく

`mysql/data`ディレクトリの中にすべて収まっているので、これを全部消せばきれいになるはず。

## MySQLコンテナに複数のDBを作成する

どうやらMySQLコンテナでは通常の方法では1つのデータベースしか作成できないっぽい。でもDumpファイルの入手時に複数データベースがあった場合、MySQLコンテナにも複数のデータベースを作成してあげないといけない。

なので、複数DBが必要なければこの項目はスキップ可。

### 新たにMySQLセッティング用ディレクトリを作成

`mysql`ディレクトリに`init.sql`というファイルを作成する。

こんな感じ。

~~~
docker
├── docker-compose.yml
├── source
|   └── test
|       ├── venderディレクトリ
|       ├── composer.json
|       ├── composer.lock
|       └── index.php
├── mysql
|   ├── dataディレクトリ
|   ├── init_data
|   |   ├── db1.dump.sql
|   |   └── db2.dump.sql
|   └── init.sql
└── php
    ├── php.ini
    └── Dockerfile
~~~

### `init.sql`の中身

以下のようにする。

~~~sql
CREATE DATABASE IF NOT EXISTS `(2個目のデータベース名)`;
GRANT ALL ON (2個目のデータベース名).* TO '(ユーザー名)'@'%';
~~~

3個、4個とある場合は繰り返せば良いと思われる。

### `docker-compose.yml`の編集

~~~yaml
services:
  mysql:
    image: mysql:(指定のバージョン)
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql  # ←これを追加
      - ./mysql/init_data:/mysql_init_data # ←これは後で使う
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

### コンテナを再起動し、DBが複数できているか確認

~~~shell
$ sudo docker-compose down
$ sudo docker-compose up -d
$ sudo docker exec -it (MySQLコンテナ名) mysql -u (ユーザー名) -p
~~~

コンテナ名は`sudo docker ps`で確認。

MySQLにログイン後以下を打つ。

~~~mysql
> show databases;
~~~

思い通りのDBができていたらOK。一旦以下を打ち、MySQLからログアウトする。

~~~mysql
> exit;
~~~

## Dumpファイルの中身をコンテナ内のDBに移す

いよいよデータのお引越しが完了する。

以下を打ち、コンテナ内に移動。

~~~shell
$ sudo docker exec -it (MySQLコンテナ名) bash
~~~

そしてコンテナ内で以下を打つ。

~~~shell
# cd mysql_init_data
# mysql -u (指定ユーザー名) -p db1 < db1.dump.sql
# mysql -u (指定ユーザー名) -p db2 < db2.dump.sql
~~~

`db1`、`db2`とかDumpファイルの名前とかは適宜読み替えること。

## 動作確認

まず扱いやすそうなテーブルを探す。レコード件数が少なかったら扱いやすいのではないだろうか、ということで各テーブルのレコード件数を見る。

一旦MySQLにログイン。

~~~shell
# mysql -u (指定ユーザー名) -p
~~~

その後以下を打つ。

~~~mysql
> select table_name, table_rows from information_schema.TABLES where table_schema = 'DB名';
~~~

そうすると指定のDB内にあるテーブルのレコード件数が一覧で表示されるので、良さげなテーブルを見つけて中身を確認する。ここでは`hogehoge`テーブルが良さげだったとする。

~~~mysql
> select * from hogehoge;
~~~

カラムとして`hoge1`と`hoge2`があったとする。

中身を確認し一旦ログアウト。

`index.php`を編集する。

~~~php+html
<?php
require('vendor/autoload.php');

use Curl\Curl;

$curl = new Curl();
$curl->get('https://www.google.com/');

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
<!-- ----------以下を変更---------- -->
$query = "SELECT hoge1 FROM hogehoge WHERE hoge2 = '(hoge2の値)';";
$result = $mysqli->query($query);
while ($row = $result->fetch_assoc()){
    $content = $row["hoge1"];
}
<!-- ----------変更ここまで---------- -->
?>
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>Hello world!</title>
</head>
<body>
    <h1>Hello world!</h1>
    <?php print(date("Y-m-d h:m:s")); ?><br>
    <?php print($content); ?><br>
    <?php if ($curl->error): ?>
        <?php print($curl->errorMessage); ?>
    <?php else: ?>
        <?php print(htmlspecialchars($curl->response)); ?>
    <?php endif; ?>
</body>
</html>
~~~

中身が出ていればOK。

[DBの出力が文字化けする場合](db_charset.html)
