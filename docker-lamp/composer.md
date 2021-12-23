# PHPのコンテナにComposerを入れてライブラリをインストール

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

