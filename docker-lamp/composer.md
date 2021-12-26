# PHPのコンテナにComposerを入れてライブラリをインストール

Composer・・・外部ライブラリの依存性を管理してくれるやつ。プロジェクトのファイルの中に`composer.json`と`composer.lock`があればComposerを使って管理しているということなので、それを使って必要なライブラリを一括でDLすればよい。

もし`composer.json`、`composer.lock`が無ければこの手順は飛ばす。

## Composerのインストール

`php/Dockerfile`を以下のように変更する。

~~~dockerfile
FROM php:(指定のバージョン)-apache
RUN apt-get update \
  && apt-get install -y \ 
    libonig-dev \
    libzip-dev \
    unzip \
    libpng-dev \
  && docker-php-ext-install \
    pdo_mysql \
    mysqli \
    mbstring \
    zip \
    gd
COPY --from=composer:(指定のバージョン) /usr/bin/composer /usr/bin/composer
WORKDIR /var/www/html
RUN composer install
~~~

Composerのバージョンなんて気にしなければ`latest`でOK。

そしてビルドし直す。

~~~shell
$ sudo docker-compose build
~~~

## `composer.json`と`composer.lock`

直属の偉い人からもらえるであろう`composer.json`と`composer.lock`を`html`ディレクトリに入れておく。こうすることでコンテナ起動時にこれら2つのファイルもコンテナ内で見えるようになる。

## Composerの実行

まずコンテナを立ち上げる。

~~~shell
$ sudo docker-compose up -d
~~~

その後に、`bash`を起動してコンテナ内で活動をする。

~~~shell
$ sudo docker exec -it (phpのコンテナ名) bash
~~~

`phpのコンテナ名`は自分で調べる。`sudo docker ps`と打てば見える。自分の場合は`docker-php-1`だった。

コマンドを実行するとBashのプロンプトが表示される。おそらく実行直後から既に`/var/www/html`に居ると思われる。一応`composer.json`と`composer.lock`があるかどうか確認する。

~~~shell
# ls
composer.json  composer.lock  index.php
~~~

そしたらおもむろに以下を打つ。

~~~shell
# composer update
~~~

そうすると`vendor`というディレクトリができて、その中にライブラリがDLされる。

~~~shell
# ls
composer.json  composer.lock  index.php  vendor
~~~

## 動作確認

`composer.json`でどんなライブラリが入るかによって違うけど、今回は例として`php-curl-class`が入ると想定する。これはPHP上でHTTPリクエストを飛ばすことができるというもの。

`composer.json`の中身はこうなっているとする。

~~~json
{
    "require": {
        "php-curl-class/php-curl-class": "(指定のバージョン)",
    }
}
~~~

`composer update`を実行した後として、`html.php`に以下を追加する。

~~~php+html
<?php
// ----------以下を追加----------
require('vendor/autoload.php');

use Curl\Curl;

$curl = new Curl();
$curl->get('https://www.google.com/');
// ----------追加ここまで----------

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
    <title>Hello world!</title>
</head>
<body>
    <h1>Hello world!</h1>
    <?php print(date("Y-m-d h:m:s")); ?><br>
    <?php print($content); ?><br>
<!-- ----------以下を追加---------- -->
    <?php if ($curl->error): ?>
        <?php print($curl->errorMessage); ?>
    <?php else: ?>
        <?php print(htmlspecialchars($curl->response)); ?>
    <?php endif; ?>
<!-- ----------追加ここまで---------- -->
</body>
</html>
~~~

Googleのトップページのソースっぽいものが表示されたら成功。

## 注意点

`composer.json`に変更を加えるごとに「Composerの実行」を毎回行わないといけない。これも`Dockerfile`や`docker-compose.yml`に書いておけば自動化されるのでは、と思うかもしれないけど、

* そもそもComposerはそんなに頻繁に実行するものではない
* `vendor`ディレクトリは一度作られてコンテナ外で永続的に保存されているのであれば、コンテナ立ち上げの時にその他のファイルと一緒にマウントすればOK

ということで、`Dockerfile`および`docker-compose.yml`には書かないこととする。
