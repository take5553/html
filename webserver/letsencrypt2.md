# Let's Encryptの証明書のドメイン変更

うっかりドメインを失効してしまい同じものが再取得できないときは違うドメインを取得することになるが、そのときにLet's Encryptの設定も変えないといけない。

ここではドメインを`arcticstreet.ddns.net`から`arctic-street.ddns.net`に変更する手順を示す。

## 環境

- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2

## 準備

Let's Encryptで証明書を取得するイメージとしては、申請したドメインで外部からちゃんとアクセスできるかどうかを確認（これをチャレンジと呼ぶ）してから証明書を発行する流れになるので、新しいドメインで外部からアクセスできるようにしておかないといけない。

* 新しいドメイン`arctic-street.ddns.net`をDDNSサービスサイトから取得する。

  （取得直後はDNSサーバーへの登録ができていない可能性があるので、数分〜１０分ほど待つ）

* ルーターのポートを開けておく。

  すでにWebサーバーを運用しているならポートは開いているはず。

## 手順

### 現在の証明書関係のファイルを削除

~~~shell
$ sudo certbot delete
~~~

このコマンドを打つと「どのドメインについての証明書を削除するか」と聞かれる。当然一つしか作っていなかったら一つしか表示されない。

画面の指示に従って削除をする。

## 新しいドメインに対する証明書を発行

~~~shell
$ sudo certbot certonly --webroot -w /home/takeshi/www/html/ -d arctic-street.ddns.net -m (使えるメールアドレス)
~~~

* `--webroot`：すでにNginxなどのWebサーバーが動いている場合に必要なオプション。逆にWebサーバーが一切動いていない場合はCertbotが持つWebサーバーを使ってチャレンジをする必要があるので、そのときはこのオプションを`--standalone`にする。
* `-w`：NginxなどWebサーバーが使用するドキュメントルート
* `-d`：発行したい新しいドメイン
* `-m`：連絡用メールアドレス

そうすると以下が出てくる。

~~~
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator webroot, Installer None
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for arctic-street.ddns.net
Using the webroot path /home/takeshi/www/html for all unmatched domains.
Waiting for verification...
Cleaning up challenges

IMPORTANT NOTES:
- Congratulations! Your certificate and chain have been saved at:
/etc/letsencrypt/live/arctic-street.ddns.net/fullchain.pem
Your key file has been saved at:
/etc/letsencrypt/live/arctic-street.ddns.net/privkey.pem
Your cert will expire on 2021-09-04. To obtain a new or tweaked
version of this certificate in the future, simply run certbot
again. To non-interactively renew *all* of your certificates, run
"certbot renew"
- If you like Certbot, please consider supporting our work by:

Donating to ISRG / Let's Encrypt:   https://letsencrypt.org/donate
Donating to EFF:                    https://eff.org/donate-le
~~~

`Congratulations!`と出ているのでうまくいったらしい。

## Nginxの設定の変更

~~~shell
$ sudo nano /etc/nginx/sites-available/default
~~~

下の方にスクロールしていけば、`# managed by Certbot`と書かれた行がたくさんあると思う。その中にある`arcticstreet.ddns.net`と書かれている部分を全て`arctic-street.ddns.net`に書き換えていけばよい。

具体的には

* `server_name`ディレクティブ
* `ssl_certificate`ディレクティブ
* `ssl_certificate_key`ディレクティブ
* リダイレクトコード`301`を返すことになっている`server`コンテキストの中の`if`文、及び`server_name`ディレクティブ

に`arcticstreet.ddns.net`の文字列があったから、それぞれ直した。

最後にNginxの再起動。

~~~shell
$ sudo nginx -s reload
~~~

