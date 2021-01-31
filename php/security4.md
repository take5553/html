# コード見直し　～セキュリティを考慮する④～　クリッキングジャック対策

対策が簡単なのでやっておく。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 対策

「クリッキングジャック」の説明については以下を参考。

[クリックジャッキングとは？その攻撃の概要と対策方法を解説](https://cybersecurity-jp.com/security-measures/6935)
[安全なウェブサイトの作り方 \- 1\.9 クリックジャッキング：IPA 独立行政法人 情報処理推進機構](https://www.ipa.go.jp/security/vuln/websecurity-HTML-1_9.html)
[Webエンジニアだったら当然知っておきたい「 クリックジャッキング対策 」とは？ \| 株式会社ヌーラボ\(Nulab inc\.\)](https://nulab.com/ja/blog/typetalk/measure-clickjacking/)

で、その対策は

~~~php
header('X-Frame-Options: DENY');
~~~

を`index.php`に入れとけばいいみたい。簡単。

`index.php`

~~~php
<?php
// CSRF対策としてセッションIDを使う
session_start();

require_once('./config/properties.php');
require_once('./model/GetFormAction.php');

// 以下を追記

// クリッキングジャック対策
header('X-Frame-Options: DENY');

// ここまで

$action = new GetFormAction();
$eventId = null;

(略)
~~~

