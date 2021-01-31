# BBSのデザインを参考にしたい

[コメントBBS](https://www.1-firststep.com/samplephp/comment-bbs-v2.0/)

![image-20210118220413299](image/design/image-20210118220413299.png)



[Simple BBS](http://www.bigcosmic.com/board/s/board.cgi?id=sampl085#del)

![image-20210118220436883](image/design/image-20210118220436883.png)



[サンプル掲示板です](https://churabbs.com/sample)

![image-20210118220728988](image/design/image-20210118220728988.png)



## セキュリティについて

* XSS
  * HTML特殊文字をエスケープ
* スパム・荒らし
  * IPで書き込み特定、禁止機能
* CSRF
  * 他サイトからのリクエストを禁止
  * トークン
  * [IPA ISEC　セキュア・プログラミング講座：Webアプリケーション編　第4章 セッション対策：リクエスト強要（CSRF）対策](https://www.ipa.go.jp/security/awareness/vendor/programmingv2/contents/301.html)
  * [これで完璧！今さら振り返る CSRF 対策と同一オリジンポリシーの基礎 \- Qiita](https://qiita.com/mpyw/items/0595f07736cfa5b1f50c)
    [とっても簡単なCSRF対策 \- Qiita](https://qiita.com/mpyw/items/8f8989f8575159ce95fc)
  * 
* 管理者モード
* まとめ
  * [【PHP初心者向け】セキュアな掲示板を最小構成から作る \- Qiita](https://qiita.com/mpyw/items/2c54d0ea95423bd88f60)
  * [YYPHP\#97「掲示板を作るときに気をつけたほうがいいセキュリティ 」「PHPセキュリティのベストプラクティス」「掲示板のいいね機能の作り方」「MVCのServiceについて聞きたい 」「大規模インフラで向いているPHPの立ち位置とは」「Laravel向けに、AWSのセキュリティガチガチの構築スクリプトを作った話」 \- Qiita](https://qiita.com/suin/items/f5032b8244ab2b432577)
  * [【8つの攻撃手法別】PHPのセキュリティ対策方法を解説！対策の重要性とは – IT業界、エンジニア、就活生、第二新卒、転職者、20代向け情報サイト](https://www.acrovision.jp/career/?p=1996)
  * [フォームとURLからの攻撃と防護策 \[PHPセキュリティー] \| WEPICKS\!](https://wepicks.net/phpsecurity-formurl/)
  * [PHP開発エンジニア必読！最低限必要なセキュリティ対策 \| Web制作会社スタイル](http://www.hp-stylelink.com/news/2013/09/20130913.php)
  * [セキュリティーを考慮したメールフォームの作り方 \- Qiita](https://qiita.com/vber_program/items/5f47dd59dcbd671aa17b)