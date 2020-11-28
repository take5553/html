# カテゴリー一覧ページを作る

カテゴリーごとに分類された記事のみ表示するようなページ。

## 環境

- ローカル（PC側）
  - Windows10
  - PowerShell 5.1
- リモート（Raspberry Pi）
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1
  - WordPress 5.5.3-ja

## 準備

そもそもカテゴリー分けするほど記事を書いていないので、カテゴリーを作り適当に記事を書く。

料理カテゴリ。

![image-20201123214643982](image/customtheme-categories/rs-image-20201123214643982.png)

トンテキ。

![image-20201123232046615](image/customtheme-categories/rs-image-20201123232046615.png)

ぶり大根。

![image-20201123232145974](image/customtheme-categories/rs-image-20201123232145974.png)

## 気付いた不備

* トップページの記事日付が変わっていない
* 個別記事ページに日付が無い
* コメント記入欄が無い（個別記事テンプレートを作るときに必要だった）
* カテゴリー「料理」へのリンクがどこにもない
* 記事のカテゴリー表示が全くない

## 作業概要

* カテゴリー一覧テンプレートを`index.php`から流用する
* 各カテゴリーへのリンクを張る
* 各記事のカテゴリーを表示する
* 作成日付を表示する

コメント欄作成は別記事にまとめる。