# CSSや画像が読み込まれない（httpsアクセス）

## 症状

動くようにはなったけれど、見た目がおかしい。明らかにCSSが読み込まれていない感じ。そういえば画像も表示されていない。

## 調査

ブラウザの検証ツールを使ってみる。（Chrome、Firefoxは`F12`キーで開く）

そうすると以下のような記述がある。

~~~
Cookie “PHPSESSID” will be soon treated as cross-site cookie against “https://localhost/xxx.css” because the scheme does not match.
~~~

なんじゃこりゃ。

これはひょっとしてCORS系か？と思ったけどちょっとだけ違うらしい。（ちなみにCORSについて[素晴らしい記事](https://qiita.com/att55/items/2154a8aad8bf1409db2b)を見つけたのでついでに勉強しておく）

### クロススキーム

[Google Developers Japan: Chrome におけるスキームフル Same-Site の適用について](https://developers-jp.googleblog.com/2020/12/chrome-same-site.html)

それっぽいことが書かれているけど、なんか違う感・・・と思ったら

> **重要な用語:** つまり、**http**://website.example のような安全でない HTTP バージョンのサイトと、**https**://website.example のような安全な HTTPS バージョンのサイトが、お互いに**クロスサイト**と見なされるということです。  

httpとhttps・・・あ！

## 原因

よく見たらサーバーには`http://localhost`のようにHTTPでアクセスしているのに、画像やCSSでは`https://localhost/xxx.css`という感じでHTTPSで取得しようとしている。

HTTPでリクエストしたページから、さらにHTTPSでリソースを取得しようとする、あるいはその逆は「クロススキーム」と呼ぶらしく、HTTPのサイトとHTTPSのサイトの関係を「クロスサイト」と呼ぶらしい。

よくわからんけどとりあえず、そういうことらしい。

## 解決法

1. PHP-ApacheコンテナにHTTPSでアクセスできるようにする。
2. ソースコードの中でHTTPSアクセスがハードコーディングされている箇所をHTTPに直す。

のどちらかで、結局HTTPに揃えるかHTTPSに揃えるかの2択。2.は各自でとしか言えないので、1.にチャレンジする。

※ちなみに自分は2.の方が早かったのと、1.でやったらブラウザから「オレオレ証明書」って指摘されたので1.が大正解とは限らないらしい。

### PHP-ApacheコンテナにHTTPSでアクセスできるようにする

参考：[Docker php-apacheを自己証明書でSSL対応（自分メモ） - Qiita](https://qiita.com/ukei2021/items/9fd5a46253f0a43f7ddb#docker-composeyml%E3%81%AE%E7%B7%A8%E9%9B%86)

#### 証明書ファイルの作成

`php`ディレクトリ以下に`ssl`ディレクトリを作成し以下のように作成していく。

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
    ├── Dockerfile
    ├── ssl.conf
    └── ssl
        ├── ssl.key
        ├── ssl.csr
        ├── san.txt
        └── ssl.crt
~~~

以下、`openssl`が必要。インストールされてるかどうかは`openssl help`と打てば何か出てくるはず。出てこなかったらインストールする。

秘密鍵作成。

~~~shell
$ openssl genrsa -out ssl.key 2048
~~~

CSR作成。色々聞かれるけど`Common Name`だけ`localhost`にしておけば大丈夫らしい。

~~~shell
$ oepnssl req -new -sha256 -key ssl.key -out ssl.csr
(色々聞かれるのでCommon Nameだけ注意)
~~~

`san.txt`はChromeで必要らしい

~~~shell
$ echo "subjectAltName = DNS:localhost" > san.txt
~~~

証明書の作成。

~~~shell
$ openssl x509 -req -sha256 -days 365 -signkey ssl.key -in ssl.csr -out ssl.crt -extfile san.txt
~~~

#### Apacheの設定

`php`ディレクトリに戻り、コンテナが立ち上がっていることを確認してから以下を打つ。

~~~shell
$ sudo docker cp (phpのコンテナ名):/etc/apache2/sites-available/default-ssl.conf ssl.conf
~~~

コンテナの中から`default-ssl.conf`を`ssl.conf`という名前にリネームしつつ取り出せるので、その中の`SSLCertificateFile`、`SSLCertificatekeyFile`をそれぞれ以下のように変更する。それ以外はそのままとする。

~~~
SSLCertificateFile    /etc/httpd/ssl/ssl.crt
SSLCertificateKeyFile /etc/httpd/ssl/ssl.key
~~~

#### `Dockerfile`の編集

以下のように変更する。

~~~dockerfile
FROM php:7.2.23-apache
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
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN a2enmod headers
#以下を追加
RUN mkdir -p /etc/httpd/ssl
RUN a2enmod ssl
COPY ./ssl.conf /etc/apache2/sites-available/ssl.conf
COPY ./ssl/ssl.key /etc/httpd/ssl/ssl.key
COPY ./ssl/ssl.crt /etc/httpd/ssl/ssl.crt
RUN a2ensite ssl
~~~

#### `docker-compose.yml`の編集

443番ポートを開けておく。

~~~yaml
services:
  mysql:
    image: mysql:(指定のバージョン)
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./mysql/init_data:/mysql_init_data
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
      - 443:443 # ←追加
    depends_on:
      - mysql
~~~

これでいけるはず。

## 動作確認

~~~shell
$ sudo docker-compose down
$ sudo docker-compose build
$ sudo docker-compose up -d
~~~

そして`https://localhost`にアクセスしてちゃんと表示されたらOK。

## これ、オレオレ証明書やでって怒られた

アクセスするとブラウザが「これ、オレオレ証明書やで」って言ってきて、警告出してきた。まあ確かにその通りだし、無視すりゃいいんだけど、なんかちょっとしょんぼり。
