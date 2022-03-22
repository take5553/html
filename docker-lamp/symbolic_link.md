# PHPの実行ファイルの場所が違う（シンボリックリンクの作成）

PHPコンテナ内のPHP本体の場所は

~~~
/usr/local/bin/php
~~~

だけど、もしソースコード内で

~~~php
$Result = shell_exec("/usr/bin/php00 -f hoge.php 2>$1");
~~~

みたいに書かれていたとしたら（`00`はPHPのバージョン番号）、ちゃんと動かなかったりする。

なぜこんなことが起こるかと言ったら、それは本番環境やテスト環境でそういう場所にPHP本体が置かれていてDockerのコンテナでは違う場所に置かれている、というだけ。

## 対処

シンボリックリンクを作ってあげよう。

### `php/Dockerfile`

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
COPY --from=composer:latest /usr/bin/composer /usr/bin/composer
RUN a2enmod headers
RUN mkdir -p /etc/httpd/ssl
RUN a2enmod ssl
COPY ./ssl.conf /etc/apache2/sites-available/ssl.conf
COPY ./ssl/ssl.key /etc/httpd/ssl/ssl.key
COPY ./ssl/ssl.crt /etc/httpd/ssl/ssl.crt
RUN a2ensite ssl
RUN curl -sSL https://github.com/mailhog/mhsendmail/releases/download/v0.2.0/mhsendmail_linux_386 -o mhsendmail \
    && chmod +x mhsendmail \
    && mv mhsendmail /usr/local/bin/mhsendmail
RUN pecl install xdebug \
  && docker-php-ext-enable xdebug
# 以下を追加
RUN ln -s /usr/local/bin/php /usr/bin/php00
~~~

これで`/usr/bin/php00`を実行してもちゃんとPHPが動く。
