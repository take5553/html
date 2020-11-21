# Nginxの設定ファイル `/etc/nginx/snippets/fastcgi-php.conf`

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1

## 設定ファイルの中身を見る前に

### NginxとPHP-FPMをインストールするとPHPが動くようになる理由

[nginx と PHP\-FPM の仕組みをちゃんと理解しながら PHP の実行環境を構築する \- Qiita](https://qiita.com/kotarella1110/items/634f6fafeb33ae0f51dc)
[nginxからphpが使えるようにPHP\-FPMをインストールする \- bnote](https://www.bnote.net/centos/php-fpm_on_nginx.html)
[NginxでPHPを動かす](https://www.spiceworks.co.jp/blog/?p=12317)

大体どれも同じようなことを書いているけど、とにかくNginxでPHPを動かすためには、PHP-FPMに取り次いでもらわないといけないらしい。

### `fastcgi_pass`について

[前回](nginx-conf2.html)保留にした`fastcgi_pass`について。

~~~
        fastcgi_pass unix:/run/php/php7.3-fpm.sock;
~~~

これはPHP-FPMのソケットの場所を指定しており、NginxとPHP-FPMがUNIXドメインソケットを通じて通信するということ。

ソケット通信、プロセス間通信とかで検索するともっと詳しい説明が得られる。

参考：
[ソケット、パイプ、共有メモリ…IPC（プロセス間通信）の最適な手段を模索してみた \| Engineer with Freedom](https://earlyfield.com/2018/11/12/post-1526/)
[Linuxのプロセス間通信 \- Qiita](https://qiita.com/MoriokaReimen/items/5c4256ef620499a88bb3)

## `/etc/nginx/snippets/fastcgi-php.conf`の中身

~~~
# regex to split $uri to $fastcgi_script_name and $fastcgi_path
fastcgi_split_path_info ^(.+?\.php)(/.*)$;

# Check that the PHP script exists before passing it
try_files $fastcgi_script_name =404;

# Bypass the fact that try_files resets $fastcgi_path_info
# see: http://trac.nginx.org/nginx/ticket/321
set $path_info $fastcgi_path_info;
fastcgi_param PATH_INFO $path_info;

fastcgi_index index.php;
include fastcgi.conf;
~~~

### `fastcgi_split_path_info`ディレクティブ

引数に正規表現が使われているから分かりにくいけど、ウチのサイトに`https://arcticstreet.ddns.net/show.php/article/0001`というURLでリクエストが飛んできた時に、

* `$fastcgi_script_name="/show.php"`
* `$fastcgi_path_info="/article/0001"`

を代入する。

#### 参考

[Module ngx\_http\_fastcgi\_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_split_path_info)
[nginx $document\_root$fastcgi\_script\_name と $request\_filename の違い \- Qiita](https://qiita.com/kotarella1110/items/3b0bd84fdb55276f37d9)

### `set`ディレクティブ

`set (変数) (値)`という書式で、変数を定義し、そこに値を入れる。

[Module ngx\_stream\_set\_module](http://nginx.org/en/docs/stream/ngx_stream_set_module.html#set)

### `fastcgi_param`ディレクティブ

PHPに渡すパラメーターの設定。

例えば上記の例では、PHP側としては`$_SERVER[ 'PATH_INFO' ]`とすれば、`"/article/0001"`という文字列が取得できる。

PHPのコードを書く人は上記の方法でリクエスト情報などを取得しているので、サーバーの設定としては必ず書かないといけないし、変えてはいけない。

[Module ngx\_http\_fastcgi\_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_param)

### `fastcgi_index`ディレクティブ

リクエストされたURLの末尾が`/`だったとき、このディレクティブで指定されているファイル名を付与して`$fastcgi_script_name`に格納する。

[Module ngx\_http\_fastcgi\_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_index)

## `/etc/nginx/fastcgi.conf`の中身

~~~
fastcgi_param  SCRIPT_FILENAME    $document_root$fastcgi_script_name;
fastcgi_param  QUERY_STRING       $query_string;
fastcgi_param  REQUEST_METHOD     $request_method;
fastcgi_param  CONTENT_TYPE       $content_type;
fastcgi_param  CONTENT_LENGTH     $content_length;

fastcgi_param  SCRIPT_NAME        $fastcgi_script_name;
fastcgi_param  REQUEST_URI        $request_uri;
fastcgi_param  DOCUMENT_URI       $document_uri;
fastcgi_param  DOCUMENT_ROOT      $document_root;
fastcgi_param  SERVER_PROTOCOL    $server_protocol;
fastcgi_param  REQUEST_SCHEME     $scheme;
fastcgi_param  HTTPS              $https if_not_empty;

fastcgi_param  GATEWAY_INTERFACE  CGI/1.1;
fastcgi_param  SERVER_SOFTWARE    nginx/$nginx_version;

fastcgi_param  REMOTE_ADDR        $remote_addr;
fastcgi_param  REMOTE_PORT        $remote_port;
fastcgi_param  SERVER_ADDR        $server_addr;
fastcgi_param  SERVER_PORT        $server_port;
fastcgi_param  SERVER_NAME        $server_name;

# PHP only, required if PHP was built with --enable-force-cgi-redirect
fastcgi_param  REDIRECT_STATUS    200;
~~~

先ほど紹介した`fastcgi_param`ディレクティブが並んでいる。

各変数の意味は以下で調べられる。

[Alphabetical index of variables](http://nginx.org/en/docs/varindex.html)

## その他

### `fastcgi_param`ディレクティブを使うタイミング

[nginx fastcgi\_params を include する箇所、割と皆間違ってるよね？ \- Qiita](https://qiita.com/kotarella1110/items/f1ad0bb40b84567cea66)

### `fastcgi.conf`と`fastcgi_params`

`/etc/nginx/fastcgi.conf`は`/etc/nginx/fastcgi_params`と中身が同じ。なぜ分かれているのか。

答えは互換性のため。

[fastcgi\_params Versus fastcgi\.conf \- Nginx Config History](https://blog.martinfjordvald.com/nginx-config-history-fastcgi_params-versus-fastcgi-conf/)