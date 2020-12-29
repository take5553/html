# URLからPHPに渡されるパラメーターについて

トラブルシューティングというか、自分の勘違いを正したので自分用にメモ。

WordPressみたいにURLに`.php`も`?`もないのにパラメーターがサーバーから勝手に渡されるという勘違い。

## 環境

リモート（Raspberry Pi）

- Raspberry Pi 3B+
- Raspberry Pi OS 10.4
- Nginx 1.14.2
- PHP 7.3.19-1~deb10u1

## 自分の勘違いと結論

勘違い

> ウチのWordPressは
>
> * `https://arcticstreet.ddns.net/wordpressblog/tonteki/`→トンテキの記事
> * `https://arcticstreet.ddns.net/wordpressblog/buri-daikon/`→ぶり大根の記事
>
> が表示されるけど、実はどちらも`https://arcticstreet.ddns.net/wordpressblog/index.php`に
>
> * `tonteki`というパラメーター（とその他もろもろのパラメーター）を渡している
>
> * `buri-daikon`というパラメーター（とその他もろもろのパラメーター）を渡している
>
> と言える。何もパラメーターを渡さなければホーム画面が表示されるというわけ。
>
> [crudmvcooptdd](crudmvcooptdd.html)

実際

`https://arcticstreet.ddns.net/wordpressblog/index.php`に

* REQUEST_URIというパラメーターとして、`/wordpressblog/tonteki/`という文字列を渡している。
* REQUEST_URIというパラメーターとして、`/wordpressblog/buri-daikon/`という文字列を渡している。
* WordPressは受け取った文字列を内部で加工してパラメーターにセットし直している
* サーバーが受け取ったURLを都合よく解釈して都合よくWordPressに渡しているわけではない

参考サイト

[WordPress の表示ロジックを理解する – Reloaded – ｜ プライム・ストラテジー株式会社](https://www.prime-strategy.co.jp/2499/)

## 勘違いの原因

[WordPressのパーマリンク設定での問題](../wordpress/permalink.html)を解決する中で、Nginxの設定ファイルに以下のように書き込んだ。

`/etc/nginx/sites-available/default`

~~~
    location /wordpressblog {
        try_files $uri $uri/ /wordpressblog/index.php?$args;
    }
~~~

この`$args`がパラメーターになっているのだろうと。

ただ、この中身についてはNginxの公式ドキュメントには

> `$args`
>
> arguments in the request line
>
> リクエストラインの中の引数
>
> [Module ngx\_http\_core\_module](https://nginx.org/en/docs/http/ngx_http_core_module.html#variables)

と書いてあるだけで実際どんな時に何が入るのかが分からない。でもWordPressはその`$args`を受け取っていて、リクエストURLに応じて表示ページを切り替えているから、きっとそういうパラメーターが入っているに違いない、と思っていた。

実際中にはクエリストリング（URLに入る`?`以降の文字列）が入っている。URLに`?`が無かったら何も入らない。

## 勘違いによって起こった問題

### 問題の発端

今回は[記事編集機能の設計](plannningedit.html)のときにページ遷移を考えて、どういうURLでその遷移を実現するかというときに、（WordPressの記事へのアクセスの仕方をイメージしながら）`bbs/edit/(id番号)/`というURLでアクセスできるようにしたい、としたのが始まり。

こういうパラメーターの引き渡し方の場合、PHPでどうやって取り出すのかと言うと、`$_SERVER['PATH_INFO']`で取り出せるとのこと。

参考：[\[PHP] リクエストパラメータ・セッションに関するまとめ \- Qiita](https://qiita.com/mpyw/items/7852213f478e8c5a2802#url%E3%81%AB%E3%83%91%E3%82%B9%E6%83%85%E5%A0%B1%E3%82%92%E5%90%AB%E3%82%81%E3%82%8B)

ただ、PHPの`$_SERVER`変数に「入ってくる」値と言うのは、サーバーが「入れている」わけで、サーバーにもそういう設定があることになる。

実際、[そういう設定がされていることは確認している](../webserver/nginx-conf3.html)。特に`$_SERVER['PATH_INFO']`の部分に関しては

> 例えば上記の例では、PHP側としては`$_SERVER[ 'PATH_INFO' ]`とすれば、`"/article/0001"`という文字列が取得できる。

と自分でも言っている。

なので、**WordPressに対して行ったNginxの設定を、掲示板に対しても行えば同じようにできるんじゃね？**と考えた。

### 問題発生

ということで、

1. 実験用にRaspberry Piのドキュメントルート内に`php-experiment`というディレクトリを作り、その中に`index.php`を以下の内容で作る。

   ~~~php
   <?php
   echo "リクエストクエリ実験<br>";
   echo "<hr>";
   echo "SERVERの中身<br>";
   foreach ($_SERVER as $key => $value) {
           if ($key == 'PHP_BBS') { continue; }
           echo "$key : $value <br>";
   }
   ~~~

   `$_SERVER`の中身を全部吐き出しつつ、DBへのログインパスワードは見えるとまずいので隠しておく。

2. Nginxの設定ファイル`/etc/nginx/sites-available/default`の中の`server`コンテキストの中に`location`ディレクティブを以下の内容で追加。

   ~~~
   	location /php-experiment {
           try_files $uri $uri/ /php-experiment/index.php?$args;
   	}
   ~~~

よしこれで`$_SERVER`の中身のうち、DBに対するパスワード以外出てくるはずだから`$_SERVER['PATH_INFO']`がどうなってるのか見てやろうじゃないか。

ブラウザから`http://192.168.1.201/php-experiment/edit/11/`と適当に打ち込みアクセス。※家の中からなのでIPでアクセスしている。

~~~
PATH_INFO :
~~~

えっ？

## Nginxの仕様

### 内部リダイレクト

Nginxの設定ファイルの`PATH_INFO`周辺を見直していたら`/etc/nginx/snippets/fastcgi-php.conf`の中に

~~~
# Bypass the fact that try_files resets $fastcgi_path_info
# see: http://trac.nginx.org/nginx/ticket/321
set $path_info $fastcgi_path_info;
fastcgi_param PATH_INFO $path_info;
~~~

> `try_files`は`$fastcgi_path_info`をリセットするから（もし使うなら）回避してね。

書いてあるURLにアクセスすると

> The [try_files](http://nginx.org/r/try_files) directive changes URI of a request to the one matched on the file system, and subsequent attempt to split the URI into $fastcgi_script_name and $fastcgi_path_info results in empty path info - as there is no path info in the URI after try_files.
>
> `try_files`はリクエストのURIを（Linuxの）ファイルシステム上のマッチしたものに書き換えて、その後の（fastcgi_split_path_infoを使った、$fastcgi_script_nameと$fastcgi_path_infoへの）URIの分割では、空のpath infoになっちゃう。だって、try_filesした後はURIにpath infoが無いもん。
>
> I don't think this should be considered as a bug, rather than a feature of how try_files work. It makes try_files not very convenient for use with fastcgi_split_path_info, but there are more than one way to workaround it, including the one provided above.
>
> これはバグじゃなくてtry_filesの仕様と考えるべきだと思う。これはtry_filesとfastcgi_split_path_infoの相性を悪くしてるけど、回避策(workaround)はいくつかあるよ。上のも含めてね。
>
> [\#321 \(try\_files & $fastcgi\_path\_info\) – nginx](https://trac.nginx.org/nginx/ticket/321)

訳はちょっと意訳。

確かに`/etc/nginx/sites-available/default`に書いた設定では

~~~
	location /php-experiment {
        try_files $uri $uri/ /php-experiment/index.php?$args;
	}
~~~

としているので、例えばブラウザで`http://192.168.1.201/php-experiment/edit/11/`にアクセスすると、

1. `try_files`で`$uri`に一致するファイルがファイルシステム上にあるかどうか検索し、無ければ`index.php?`に`$args`を結合させて内部リダイレクトする。

   * アクセスしたURLに`?`が無い限り`$args = ""`
   * `$uri = /php-experiment/edit/11/`だったのが、内部リダイレクトにより`$uri = /php-experiment/index.php`に書き変わる。

2. `*.php`にアクセスするという形になるので、`location ~ \.php$`ディレクティブが反応する。

   ~~~
   location ~ \.php$ {
       include snippets/fastcgi-php.conf;
       fastcgi_pass unix:/run/php/php7.3-fpm.sock;
   }
   ~~~

3. 以下の設定ファイルを順番に読み込む

   1. `/etc/nginx/snippets/fastcgi-php.conf`
      * このタイミングで`fastcgi_split_path_info`ディレクティブが実行される。
      * ただし`$uri`は内部リダイレクトにより書き変わっているので想定通りの動きをしない。
   2. `/etc/nginx/fastcgi.conf`

4. PHP-FPMと通信をする。（おそらくパラメーターも渡している）

ということが起こる。

### `fastcgi_split_path_info`ディレクティブ

色々実験する中で分かったけど、そもそも`fastcgi_split_path_info`は`$uri`の中に`.php`という文字列が見つからない場合、`$fastcgi_path_info`に何も入れないということが分かった。

## 結論

* URLに`.php`も`?`も入れたくない場合
  * `$_SERVER['REQUEST_URI']`から文字列を受け取って頑張る。

* URLに`.php`が入っても良いけど`?`は入れたくない場合

  * `$_SERVER['REQUEST_URI']`から文字列を受け取って頑張る。

  * サーバーの内部リダイレクトしないように設定すれば`$_SERVER['PATH_INFO']`でパラメーターが受け取れる。

* URLに`?`が入っていい場合

  * `$_GET[パラメーター名] = 値`としてクエリストリングからパラメーター取り出す。
  * サーバーの内部リダイレクトしないように設定すれば`.php`と`?`の間の文字列が`$_SERVER['PATH_INFO']`で受け取れる。

## ついでに分かったこと

`/etc/nginx/sites-available/default`に以下のように記述して、`/php-experiment/`にアクセスしたとしても`$_SERVER['URI_BEFORE_REDIRECT']`は定義されないので注意。

~~~
location ~ \.php$ {
	fastcgi_param URI_AFTER_REDIRECT $uri;
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/run/php/php7.3-fpm.sock;
}

location /php-experiment {
	fastcgi_param URI_BEFORE_REDIRECT $uri;
	try_files $uri $uri/ /php-experiment/index.php?$args;
}
~~~

内部リダイレクトが起こったら、リダイレクト前に設定したサーバー環境変数は引き継がれないっぽい。

設定を引き継ごうと思ったら一旦Nginxの変数に入れると良い。

~~~
location ~ \.php$ {
	fastcgi_param URI_BEFORE_REDIRECT $uri_before_redirect;
	fastcgi_param URI_AFTER_REDIRECT $uri;
	include snippets/fastcgi-php.conf;
	fastcgi_pass unix:/run/php/php7.3-fpm.sock;
}

location /php-experiment {
	set $uri_before_redirect $uri;
	try_files $uri $uri/ /php-experiment/index.php?$args;
}
~~~

## 参考

PHPのリクエストパラメーター
[\[PHP] リクエストパラメータ・セッションに関するまとめ \- Qiita](https://qiita.com/mpyw/items/7852213f478e8c5a2802#url%E3%81%AB%E3%83%91%E3%82%B9%E6%83%85%E5%A0%B1%E3%82%92%E5%90%AB%E3%82%81%E3%82%8B)
[PHPでURLを取得する（部分取得やクエリ文字除外なども）](https://www.flatflag.nir87.com/url-963)
[そのリクエストパラメータ、クエリストリングに入れますか、それともボディに入れますか \- Qiita](https://qiita.com/sakuraya/items/6f1030279a747bcce648)

PHPの`$_SERVER`
[PHP: $\_SERVER \- Manual](https://www.php.net/manual/ja/reserved.variables.server.php)

`$_SERVER['PATH_INFO']`が出力されない
[\#321 \(try\_files & $fastcgi\_path\_info\) – nginx](https://trac.nginx.org/nginx/ticket/321)
[Problem with fastcgi\_split\_path\_info on ubuntu precise](https://forum.nginx.org/read.php?2,238825,238825#msg-238825)
[php \- I do not have PATH\_INFO in $ \_SERVER \- Stack Overflow](https://stackoverflow.com/questions/17635596/i-do-not-have-path-info-in-server)
[pathinfo \- What exactly is PATH\_INFO in PHP? \- Stack Overflow](https://stackoverflow.com/questions/2261951/what-exactly-is-path-info-in-php)
[php \- PATH\_INFO in $\_SERVER always empty \- NGINX \+ FPM 7\.3 \+ Ubuntu 18\.04 \- Stack Overflow](https://stackoverflow.com/questions/56640592/path-info-in-server-always-empty-nginx-fpm-7-3-ubuntu-18-04)

Nginxの`fastcgi_split_path_info`ディレクティブについて
[Module ngx\_http\_fastcgi\_module](http://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_split_path_info)

Nginxの内部リダイレクトについて
[nginx連載5回目: nginxの設定、その3 \- locationディレクティブ \- インフラエンジニアway \- Powered by HEARTBEATS](https://heartbeats.jp/hbblog/2012/04/nginx05.html)
[nginx の php\-fpm で index\.php のフロントコントローラーありきの設定 \- ngyukiの日記](https://ngyuki.hatenablog.com/entry/2020/09/04/004445)

