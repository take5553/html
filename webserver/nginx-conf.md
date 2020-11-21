# Nginxの設定ファイル `/etc/nginx/nginx.conf`

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1

## 設定ファイルの場所

ファイル名は`nginx.conf`。

場所は、環境とかインストール方法によって場所が変わるらしい。

* `/etc/nginx`
* `/usr/local/nginx/conf`
* `/usr/local/etc/nginx`

など。

自分は`/etc/nginx`にあった。

## 設定ファイルの中身

開いてみる。

~~~shell
$ cat /etc/nginx/nginx.conf
~~~

中身はこちら。

~~~
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
        # multi_accept on;
}

http {

        ##
        # Basic Settings
        ##

        sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;
        # server_tokens off;

        # server_names_hash_bucket_size 64;
        # server_name_in_redirect off;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ##
        # SSL Settings
        ##

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

        ##
        # Logging Settings
        ##

        access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;

        ##
        # Gzip Settings
        ##

        gzip on;

        # gzip_vary on;
        # gzip_proxied any;
        # gzip_comp_level 6;
        # gzip_buffers 16 8k;
        # gzip_http_version 1.1;
        # gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

        ##
        # Virtual Host Configs
        ##

        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}


#mail {
#       # See sample authentication script at:
#       # http://wiki.nginx.org/ImapAuthenticateWithApachePhpScript
#
#       # auth_http localhost/auth.php;
#       # pop3_capabilities "TOP" "USER";
#       # imap_capabilities "IMAP4rev1" "UIDPLUS";
#
#       server {
#               listen     localhost:110;
#               protocol   pop3;
#               proxy      on;
#       }
#
#       server {
#               listen     localhost:143;
#               protocol   imap;
#               proxy      on;
#       }
#}
~~~

`#`で始まる部分はコメントなので無視してよいので、省略した形が以下。

~~~
user www-data;
worker_processes auto;
pid /run/nginx.pid;
include /etc/nginx/modules-enabled/*.conf;

events {
        worker_connections 768;
}

http {

		sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;

        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;

		access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
        
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
}
~~~

### シンプルディレクティブとブロックディレクティブ

#### シンプルディレクティブ

シンプルディレクティブとは、1行で終わるような、名前とパラメーターが空白で分けられて終わりがセミコロンになっているもの。例えば以下は`worker process`の実行ユーザーが`www-data`ですよ、ということを表すらしい。`worker process`とは、実際にHTMLファイルなどを読み込みレスポンスとして返すプロセス。なので、こいつの実行ユーザー`www-data`が、HTMLファイルなどを読めるように権限を設定してあげないといけない。

~~~
user www-data;
~~~

次は`worker process`の数。`auto`とすると多分適当にマルチスレッドでリクエストを処理してくれる。多分。

~~~
worker_processes auto;
~~~

以下は、PIDが記されたファイルはどれですか、という設定。Nginxを停止させるときに、ここに書かれているPIDを取得し、そいつを`kill`コマンドで停止させるんでしょう。多分。完全に想像。

~~~
pid /run/nginx.pid;
~~~

#### ブロックディレクティブ

以下の様なもの。これは`events`ディレクティブといい、その中にさらにブロックディレクティブを入れ子にすることができ、そういう場合はコンテキストと呼ばれるらしい。

~~~
events {
        worker_connections 768;
}
~~~

ちなみに一番外側にあるディレクティブは「`main`コンテキストに属する」と言うらしい。上のは「`main`コンテキストに属する`events`コンテキスト」と呼ぶらしい。でも`events`ディレクティブと呼んでるサイトもある。分かればどっちでもよい。

#### `include`ディレクティブ

これだけちょっと特殊で、`include /etc/nginx/modules-enabled/*.conf;`と書かれているのは`/etc/nginx/modules-enabled`の中にある`.conf`という設定ファイルも読み込みなさいよ、ということ。要は`nginx.conf`以外にも設定ファイルがあるよということ。

### `events`コンテキスト

~~~
events {
    worker_connections 1024;
}
~~~

1つの`worker process`が処理できるコネクション数らしい。詳しくは不明。このままで良い。

### `http`コンテキスト

Webサーバーとしての設定はこの`http`コンテキストの中に書くらしい。

以下は、アマチュアレベルでは知る必要の無さそうなディレクティブたち。解説しているサイトを読んだけど、別に調整することは今後無さそう。

~~~
		sendfile on;
        tcp_nopush on;
        tcp_nodelay on;
        keepalive_timeout 65;
        types_hash_max_size 2048;

        include /etc/nginx/mime.types;
        default_type application/octet-stream;
~~~

解説サイト：
[nginxの設定ファイル nginx\.conf の読み方 超入門 – 或る阿呆の記](https://hack-le.com/nginx/)
[ngx\_http\_core\_module モジュール 日本語訳](http://mogile.web.fc2.com/nginx/http/ngx_http_core_module.html)

---

以下もSSL関連だということはわかるけど、いじる必要は無さそう。

~~~
        ssl_protocols TLSv1 TLSv1.1 TLSv1.2; # Dropping SSLv3, ref: POODLE
        ssl_prefer_server_ciphers on;
~~~

[ngx\_http\_ssl\_module モジュール 日本語訳(ssl_protocols)](http://mogile.web.fc2.com/nginx/http/ngx_http_ssl_module.html#ssl_protocols)
[ngx\_http\_ssl\_module モジュール 日本語訳(ssl_prefer_server_ciphers)](http://mogile.web.fc2.com/nginx/http/ngx_http_ssl_module.html#ssl_prefer_server_ciphers)

---

以下はアクセスログとエラーログの場所。読めば分かる。

~~~
		access_log /var/log/nginx/access.log;
        error_log /var/log/nginx/error.log;
~~~

---

これはgzip圧縮をしてレスポンスを送信するかどうか。

~~~
        gzip on;
~~~

有効にしていると[BREACH攻撃](https://blog.ohgaki.net/breach-attack-explained-why-and-how)を受ける可能性があるとのこと。次に見る`/etc/nginx/sites-available/default`の中にも「OFFにしとけ」という記述があるのでOFFにしておく。

ちなみにWebページ上にパスワードやクレジットカード番号など秘密にしておかないといけない情報を載せると危ない。そんなことはしないと思うけど一応そうしておこうか。

~~~
        gzip off;
        # 変更後は sudo nginx -s reload でNginxを再起動
~~~

---

その他の設定は別ファイルにまとめてるからそっちを見てね。

~~~
        include /etc/nginx/conf.d/*.conf;
        include /etc/nginx/sites-enabled/*;
~~~

でも`/etc/nginx/conf.d`の中には何もなかった。`/etc/nginx/sites-enabled`の中にはシンボリックリンクが一つだけ。リンク先は`/etc/nginx/sites-available/default`。なのでそれを追いかければ良さそう。

[続く](nginx-conf2.html)。

