# DockerでLAMP編

実行環境の構築として、ローカルにLAMP環境を組まないといけないことがあると思う。これをDockerでバージョンを指定しながら組めると楽なのではないか。

以下、実務で環境を指定されたものとして進める。

## 構築編

1. [とにかくPHPとMySQLを動かす](install.html)
2. [PHPのコンテナにComposerを入れてライブラリをインストール](composer.html)
3. [DBをコピーする](copy_db.html)

## 調整編

以後、プロジェクトのソースコードをドキュメントルートに移してから完動するまで。

1. [Internal Server Errorが出た（Apacheのモジュール有効化）](apache_module.html)
2. [CSSや画像が読み込まれない（httpsアクセス）](http_https.html)
2. [DBから取得した日本語文字列が文字化けする（DBの文字セットを指定）](db_charset.html)
2. [メールが送信できない（メール送信機能テスト）](sendmail.html)
2. [デバッグ機能を準備する（VSCode + XDebug）](php_debug.html)
2. [PHPのエラーがHTML上に表示されてしまう（PHPのエラーレベル設定）](disable_notice.html)
2. 
