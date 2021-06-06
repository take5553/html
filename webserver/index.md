# Webサーバー立ち上げ編

Webサーバー（Nginx）を導入し、HTMLファイルを作ってアップロードする。

- Webサーバー稼働
  - [Webサーバー(Nginx)をインストールし、HTMLファイルを外部に公開](nginx.html)
  - [URLを`https://`でもアクセスできるようにする（Let's Encrypt使用）](letsencrypt.html)
    * 使用ドメインが変わった→[Let's Encryptの証明書のドメイン変更](letsencrypt2.html)
- HTMLファイル作成
  - [HTMLファイル作成ツールとしてのTyporaの紹介とMarkdown記法について](typora.html)
  - [Typoraのテーマについて](typoratheme.html)
  - [HTMLファイル作成例](htmlexample.html)
- HTMLファイルをRaspberry Piにアップロード
  - [HTML更新（手動）](update1.html)
  - [HTML更新（D&Dでアップロード）](update2.html)
  - [HTML更新（ダブルクリックでフォルダごとアップロード）](sync.html)
  - [HTML更新（リモートにないディレクトリを検知して自動で作成）](sync2.html)
  - [HTML更新（Gitを使う）](syncgit.html)
- Nginxの操作・設定
  * [Nginxを操作するコマンド](nginx-command.html)
  * Nginxの設定ファイル
    * [`/etc/nginx/nginx.conf`](nginx-conf.html)（最初に読まれる設定）
    * [`/etc/nginx/sites-available/default`](nginx-conf2.html)（バーチャルサーバー関連）
    * [`/etc/nginx/snippets/fastcgi-php.conf`](nginx-conf3.html)（PHP関連）

