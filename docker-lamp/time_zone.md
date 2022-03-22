# 表示される時刻がおかしい（タイムゾーンの設定）

むしろ今までタイムゾーンを設定していなかったのがおかしい。

## 対処

### PHPコンテナの`Dockerfile`

~~~dockerfile
FROM php:(指定のバージョン)-apache
RUN apt-get update \
  && apt-get install -y \
    libonig-dev \
    libzip-dev \
    unzip \
    libpng-dev \
    tzdata \ # ←追加
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
RUN ln -s /usr/local/bin/php /usr/bin/php00
# 以下を追加
ENV TZ Asia/Tokyo
~~~

### MySQLコンテナの`Dockerfile`

~~~dockerfile
FROM mysql:(指定のバージョン)
RUN apt-get update \ #←aptからapt-getに変更
    && apt-get install -y locales-all tzdata #←aptからapt-getに変更してtzdataを追加
ENV LANG ja_JP.UTF-8
ENV LANGUAGE ja_JP:ja
ENV LC_ALL ja_JP.UTF-8
# 以下を追加
ENV TZ Asia/Tokyo
~~~

なぜか`apt`のままだとエラーが返ってきた。謎。ひょっとしたらコンテナのバージョンによっては`apt`でもいけるかもしれない。
