# VSCodeの拡張機能①「PHP Intelephense」とPHP書初め

[WordPress編](../wordpress/)ではさもPHP知ってるぜ的な雰囲気を出しながら書いてたけど、PHPコードを一から書き起こすのはすごい久しぶりなので、どういう風に書けばいいのか思い出す。

コーディングサポートをしてくれる拡張機能「PHP Intelephense」の使い方も一緒に見ていく。

## 環境

* ローカル
  * Windows 10
  * VSCode 1.51.1
  * PHP 8.0.0

## PHP Intelephense

![image-20201206181113058](image/startphp/rs-image-20201206181113058.png)

なんか「要php-language-server」と言っているサイトもあるけど、先にインストールするべきものが見当たらないので、インストールボタンをぽちっとな。特に問題なくインストールされた。

説明書きに「拡張機能の検索欄に`@builtin php`と打ち、`PHP Language Features`を無効にせよ（`PHP Language Basics`はそのままでいい）」と書いてあるからそうする。

もう一つ、「標準的なPHPファイルの拡張子ではないファイル（例えば`.module`とか）にもPHPコーディングサポートを適用したかったら設定の`files.associations`に追加してね」って書いてあるけど、とりあえずは不要なので放置。

そしてVSCodeを再起動。

これで拡張機能がインストールされたということか。

## 何か書く

`index.php`にPHPコードを書く。

![image-20201206200122276](image/startphp/rs-image-20201206200122276.png)

「e」と入力するだけで候補がずらっと出てきた。しかもPHPコードとしては不完全な状態なので左のエクスプローラーの`index.php`が赤い。

試しにPHP Intelephenseを無効にして再起動して同じことをしてみる。

![image-20201206200511723](image/startphp/rs-image-20201206200511723.png)

出てくる候補が変わった。`echo`を打ちたいのにこれでは最後まで打たないといけない。

再度PHP Intelephenseを有効化して適当に最後までコードを打ってみる。

![image-20201206200820652](image/startphp/rs-image-20201206200820652.png)

ほほう。

これをブラウザで表示させると

![image-20201206201119492](image/startphp/image-20201206201119492.png)

ほほう。

組み込み関数を入れるとヒントが表示される。これはいいね。

![image-20201206201605346](image/startphp/image-20201206201605346.png)

自作関数もちゃんと定義した通りにヒントを出してくれる。

![image-20201206202423286](image/startphp/image-20201206202423286.png)

なるほど。