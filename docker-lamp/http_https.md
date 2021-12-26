# CSSや画像が読み込まれない（httpsとhttp）

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
2. ソースコードの中でHTTPSアクセスがハードコーディングされている箇所を潰す。

のどちらかで、結局HTTPに揃えるかHTTPSに揃えるかの2択。2.は各自でとしか言えないので、1.にチャレンジする。

### PHP-ApacheコンテナにHTTPSでアクセスできるようにする

