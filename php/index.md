# PHP（掲示板作成）編

掲示板システムを作成する。VSCodeを使って開発していく。

* 準備
  * [ワークフォルダ作成、VSCode準備、Gitリポジトリ（ローカル）作成](preparation.html)
  * [GitHubにリモートリポジトリ作成、XAMPPインストール](preparation2.html)
  
* VSCodeに慣れる

  * [VSCodeの拡張機能①「PHP Intelephense」とPHP書初め](startphp.html)
  * [VSCodeの拡張機能②「PHP Debug」でデバッグをしてみる](debug.html)
  * [VSCodeの拡張機能③「php cs fixer」でコードの整形](fixcode.html)
  * [VSCodeの拡張機能④「PHP DocBlocker」でコードの説明を書く](docblock.html)
  * [VSCodeの拡張機能⑤「その他いろいろ」](otherextensions.html)
  
* コーディング準備
  * [Emmetを使ってHTMLの準備](htmlform.html)
  * [データベースの準備と動作確認](makedb.html)
  * [Raspberry Pi上でもデータベースが動くことを確認](onraspberrypi.html)
  * [GitHubからRaspberry PiにPullする処理を自動化](autopull.html)
  * [PHPUnitでテストを試し書き](phpunit.html)
    * [デバッグでテスト失敗を乗り越える](testanddebug.html)
    * [テストの世界](testcoverage.html)
  * [CRUDとMVCとOOPとTDD、そして今回の開発方針](crudmvcooptdd.html)
  * [postsテーブルの修正](fixpoststable.html)
  
* 実装

  * 投稿
    * [投稿機能の追加](post.html)
    * [投稿機能のテストを書く](posttest.html)
  * 記事表示
    * [記事表示機能の追加](getposts.html)
    * [記事表示のテストを書く](getpoststest.html)
  * 記事編集
    * [記事編集機能の設計](planningedit.html)
    * [記事編集機能のテストを書く](edittest.html)
    * [記事編集機能の追加](edit.html)
    
    * [記事編集機能をビューに組み込んで統合テスト](editinview.html)
  * 削除
    * [記事削除機能の設計](planningdelete.html)
    * [記事削除機能のテストを書く](deletetest.html)
    * [記事削除機能の追加](delete.html)
    * [記事削除機能をビューに組み込んで統合テスト](deleteinview.html)
  * コード見直し
    * 重複している処理を整理
    * 特殊文字のエスケープ
    * オブジェクト指向でコード整理を試みる

* トラブルシューティング
  
  * [VSCode上でPHP Debugが動かない問題](troubleshooting1.html)
  * [PHPファイルにDBへのログインパスワードを直書きしないようにする](hidepassword.html)
  * [URLからPHPに渡されるパラメーターについて](aboutquerystrings.html)

