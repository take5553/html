# コンテナ内で日本語を打ち込むとすぐ消える

しばらく使っていると、特にMySQLで色々やるときに日本語が使えないことに気づく。

日本語を打ち込んだすぐそばから消えるみたいな。2バイト文字は文字として認識されない感がある。

※MySQLコンテナに限った話ではないので、PHPコンテナでも問題が起こるなら同様の対処で解決できるはず。（未検証

## 解決策

### 概要

MySQLコンテナ内で`locales-all`というパッケージをインストールして以下の環境変数を設定する。

~~~
LANG=ja_JP.UTF-8
LANGUAGE=ja_JP:ja
LC_ALL=ja_JP.UTF-8
~~~

`locales-all`じゃなくて`locales`でもOKとかいう情報もあるけど、自分はできなかった。（主に`LC_ALL`の設定でエラーが出た）

### やり方

#### MySQLコンテナ用`Dockerfile`の配置

コンテナビルド時にパッケージのインストールが必要なので、MySQL用コンテナの`Dockerfile`を作る。

`mysql`ディレクトリに直接置くと同階層にあるファイルやディレクトリをコンテクストとしてDockerデーモンに送ってしまうため、一度構築した後に再度`build`するようなことがあると`data`ディレクトリの内容が重かったらめちゃめちゃ遅くなる。

```
docker
├── docker-compose.yml
├── source
|   └── test
|       ├── venderディレクトリ
|       ├── composer.json
|       ├── composer.lock
|       └── index.php
├── mysql
|   ├── dockerfile       ←追加
|   |   └── Dockerfile   ←追加
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
```

そういう意味では`php`ディレクトリの中の`Dockerfile`も場所が悪いけど、まあこっちは同階層のデータが重くなるということは無さそうなので放置。気になったら後で直しておく。

#### `docker-compose.yml`の編集

```yaml
services:
  mysql:
    build: ./mysql/dockerfile  # ←この行を変更
    volumes:
      - ./mysql/data:/var/lib/mysql
      - ./mysql/init.sql:/docker-entrypoint-initdb.d/init.sql
      - ./mysql/init_data:/mysql_init_data
      - ./mysql/my.cnf:/etc/mysql/conf.d/my.cnf
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
      - ./php/log:/var/log
      - ./source/test:/var/www/html
    ports:
      - 80:80
      - 443:443
    depends_on:
      - mysql
  mailhog:
    image: mailhog/mailhog
    ports:
      - 1025:1025
      - 8025:8025
```

#### MySQLコンテナ用の`Dockerfile`の編集

~~~dockerfile
FROM mysql:(指定のバージョン)
RUN apt update \
    && apt install -y locales-all
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
~~~

#### 再ビルド

~~~shell
$ sudo docker-compose build
~~~

## 参考

[DockerのMySQLコンテナを日本語対応させる - Qiita](https://qiita.com/zongxiaojie/items/6b593ec4ce5e85bb342c)
