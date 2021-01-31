# コード見直し　～セキュリティを考慮する③～　CSRF対策

今のままだと`index.php`に`POST`メソッドでリクエストを飛ばせば、リクエスト送信元は問題にならない。別サイトからでも良い。

これはCSRF（クロスサイトリクエストフォージェリ）という脆弱性を含むことになる。これによる危険性はググってもらうとして、これの対策をする。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 対策方法

### セッションIDを使う

リクエスト送信元を`https://arcticstreet.ddns.net/bbs/index.php`に限定すればよい。

ただし、`$_SERVER['HTTP_REFERER']`（ページ遷移元）をチェックすればいいという話ではない。というのも基本的にはHTTPリクエストというのは編集可能で、攻撃者がその気になれば自由に書き換えることができる。

ここではよくある方法で、セッションIDを使って確認する方法を取る。

参考（CSRF対策）
[【8つの攻撃手法別】PHPのセキュリティ対策方法を解説！対策の重要性とは – IT業界、エンジニア、就活生、第二新卒、転職者、20代向け情報サイト](https://www.acrovision.jp/career/?p=1996)
[フォームとURLからの攻撃と防護策 \[PHPセキュリティー] \| WEPICKS\!](https://wepicks.net/phpsecurity-formurl/)
[【PHP初心者向け】セキュアな掲示板を最小構成から作る \- Qiita](https://qiita.com/mpyw/items/2c54d0ea95423bd88f60#csrf%E8%84%86%E5%BC%B1%E6%80%A7%E5%AF%BE%E7%AD%96)

参考（セッション）
[PHP: Sessions \- Manual](https://www.php.net/manual/ja/book.session.php)
[【PHP超入門】Cookieとセッションについて \- Qiita](https://qiita.com/7968/items/ce03feb17c8eaa6e4672)

簡単に説明すると

1. 掲示板にアクセスしたときに、サーバーが発行するユーザーごとの一意のIDをHTTPレスポンスとしてクライアントに送信（と同時にサーバーにもそのIDを一時保存）
2. ユーザーが記事作成
3. 記事を投稿するときにそのIDも一緒にサーバーに送る
4. サーバーは保存されているIDとクライアントから送られてきたIDを照合し、一致した場合だけアクションする

という感じ。

こうすることにより

1. 記事を投稿するにはまずIDを発行してもらわないといけない
2. IDを発行してもらうためには投稿者が`index.php`にアクセスしないといけない。

となり、普通はそのまま`index.php`から記事投稿することになる。

厳密に言えばそのIDを抜き取れば結局はリクエスト送信元はどこでもいいことになるけど、そのIDは取得した人（正確には取得したクライアント）しか使えないため、不特定の他人に意図しない投稿（例えば殺人予告とか）をさせることはできない。

自分ならリクエスト送信元を別の場所に変えることができるけど、別にわざわざそんなことせずに`index.php`から投稿すりゃいいじゃん、という話。

### ただし、筆が遅い人のための救済措置が必要

普通に掲示板に何を書くか考えすぎて24分以上経ってしまって、いざ投稿ボタンを押したときに

「セッション切れです。再度投稿してください」

とかいう表示とともに入力欄が真っ白になっていたら怒りを覚えるので、そういう人たちのための救済措置を講じる。

※今回この措置のせいで変更がややこしくなってしまった。

## 実装

### セッションID発行

`index.php`

~~~php
<?php
// CSRF対策としてセッションIDを使う
session_start();

require_once('./config/properties.php');
require_once('./model/GetFormAction.php');

(略)
~~~

これで一意のIDが発行されサーバーとクライアントに保存される。

### 投稿時にセッションIDを送る

`view/post.php`

`token`という名前の隠しパラメーターを設定。

~~~php+HTML
        <p>
            <input type="hidden" name="eventId" value="save">
            
            <!-- 以下を追記 -->
            <input type="hidden" name="token"
                   value="<?php echo htmlentities(password_hash(session_id(), PASSWORD_DEFAULT), ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
            <!-- ここまで -->
            
            <input type="submit" value="送信">
        </p>
    </form>
</div>
<!-- 記事入力エリア終了 -->
~~~

`password_hash`でハッシュ化しているのはやりすぎかもしれないけど、一応念のため。

`view/edit.php`

~~~php+HTML
        <p>
            <input type="hidden" name="id"
                   value="<?php echo htmlentities($edit_data['id'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
            
            <!-- 以下を追記 -->
            <input type="hidden" name="token"
                   value="<?php echo htmlentities(password_hash(session_id(), PASSWORD_DEFAULT), ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
            <!-- ここまで -->
            
            <button name="eventId" value="update">更新</button><br><br>
            <button name="eventId" value="delete">削除</button>
        </p>
    </form>
</div>
<!-- 記事入力エリア終了 -->
~~~

### 送られてきたセッションIDを確認

`index.php`

~~~php
<?php
// CSRF対策としてセッションIDを使う
session_start();

require_once('./config/properties.php');
require_once('./model/GetFormAction.php');

$action = new GetFormAction();
$eventId = null;

// 以下を追記

$repost_flag = false;
$reupdate_flag = isset($_SESSION['reupdate_flag']) ? $_SESSION['reupdate_flag'] : false;

$filtered_post_data['id'] = (string)filter_input(INPUT_POST, 'id');
$filtered_post_data['name'] = (string)filter_input(INPUT_POST, 'name');
$filtered_post_data['email'] = (string)filter_input(INPUT_POST, 'email');
$filtered_post_data['body'] = (string)filter_input(INPUT_POST, 'body');
$filtered_post_data['password'] = (string)filter_input(INPUT_POST, 'password');
$token = (string)filter_input(INPUT_POST, 'token');
$params = $action->GetParam();

// イベントIDを取得
// CSRF対策としてpost.phpおよびedit.phpに埋め込んだトークンが一致しなければアクションは起こさない
if ($_SERVER['REQUEST_METHOD'] === 'POST' && isset($_POST['eventId'])) {
    if (password_verify(session_id(), $token)) {
        $eventId = $_POST['eventId'];
    } else {
        $errmsg = "セッションエラー。";
        // もし投稿だったり更新だったりしたときに、内容が失われると可哀そうだからそれだけ救ってあげる
        if ($_POST['eventId'] === 'update' && ctype_digit($_POST['id'])) {

            // 必要な情報をセッションに持たせてリダイレクトする（パスワード以外）
            $_SESSION['name'] = $filtered_post_data['name'];
            $_SESSION['email'] = $filtered_post_data['email'];
            $_SESSION['body'] = $filtered_post_data['body'];
            $_SESSION['reupdate_flag'] = true;
            $_SESSION['errmsg'] = $errmsg . "再度更新してください。";

            $url = "/bbs/edit/" . $filtered_post_data['id'];
            header('Location: ' . $url, true, 302);
            exit;
        } elseif ($_POST['eventId'] === 'save') {
            $repost_flag = true;
            $errmsg .= "再度投稿してください。";
        }
    }
}

// ここまで

switch ($eventId) {
    case 'save':
        $saveResult = $action->SaveDBPostData($_POST);
        require('./view/post.php');
        break;

(略)
~~~

最初の構想では`password_verify(session_id(), $token)`が`false`になったら`$eventId = null`でいいかなーと思ったけど、救済措置のために色々ややこしくなってしまった。

セッションが切れてしまった場合、`$_POST`は存在しているはずなのでそれを再利用する。`(string)filter_input(INPUT_POST, 'キー名')`は`$_POST['キー名']`に値が格納されていてそれが文字列かどうかを確認し、そうなら中の値を取り出すという処理。`(string)`としてキャストすることで、定義されていない場合、空文字が格納され、`$filtered_post_data`は確実に定義されることになる。

本当は`$_POST`からデータを受け取るときはこうした方がいいらしい。またあとでコードを見直しておこう。

単なる記事投稿ならそのまま`$_POST`の内容をフォームに入れてあげたらいいんだけど、記事編集モードでもうんうん悩んで24分経っちゃったときの処理はリダイレクトが絡むので、セッションに保存することにした。でもここはCookieでも良かったかな。

### 記事編集時のみセッションからデータを取得

`index.php`

~~~php
switch ($eventId) {

        (略)
        
    default:
        switch ($params['mode']) {
            case 'edit':
                $edit_data = $action->GetDBOnePostData($params['id']);
                if (is_array($edit_data)) {
                    
                    // 以下を追記
                    
                    // $edit_dataが配列かどうか確認してからセッション切れ救済の処理をしているのは、
                    // 渡された記事IDが、未削除の記事を指定していないか確認したいから。
                    if ($reupdate_flag) {
                        $edit_data['name'] = $_SESSION['name'];
                        $edit_data['email'] = $_SESSION['email'];
                        $edit_data['body'] = $_SESSION['body'];
                        $errmsg = $_SESSION['errmsg'];
                    }
                    
                    // ここまで
                    
                    require('./view/edit.php');
                } else {
                    require('./view/id_error.php');
                }
                break;
            default:
                require('./view/post.php');
                break;
        }
        break;
}
~~~

`$edit_data`に入れておけばViewが配置してくれる。

### セッション切れの救済措置をViewに反映

`view/post.php`

もし`$repost_flag`が立っていたら、入力欄に再格納してあげる。

~~~php+HTML
(略)

	<!-- 記事入力エリア -->
    <div class="input_area">
        <form action="./index.php" method="post" id="post_form">
            <p>
                名前：<br>
                <input type="text" name="name" id="name" value="<?php if ($repost_flag) {
    echo htmlentities($filtered_post_data['name'], ENT_HTML5 | ENT_QUOTES, "UTF-8");
}?>">
            </p>
            <p>
                メールアドレス：<br>
                <input type="email" name="email" id="email" value="<?php if ($repost_flag) {
    echo htmlentities($filtered_post_data['email'], ENT_HTML5 | ENT_QUOTES, "UTF-8");
}?>">
            </p>
            <p>
                本文：<br>
                <textarea name="body" id="body" cols="30" rows="10"><?php if ($repost_flag) {
    echo htmlentities($filtered_post_data['body'], ENT_HTML5 | ENT_QUOTES, "UTF-8");
}?></textarea>
                
(略)
~~~

### エラーメッセージ表示

`view/post.php`

~~~php+HTML
    <!-- エラーメッセージ表示エリア -->
    <?php if (isset($saveResult) && $saveResult == false) :?>
    <div class="errormsg">
        <p>記事投稿に失敗しました。</p>
    </div>
    <?php elseif (isset($updateResult) && $updateResult == false) :?>
    <div class="errormsg">
        <p>記事編集に失敗しました。</p>
    </div>

	<!-- 以下を追記 -->

    <?php elseif (isset($errmsg)) :?>
    <div class="errormsg">
        <p><?php echo htmlentities($errmsg, ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>
        </p>
    </div>

	<!-- ここまで -->

    <?php endif; ?>
    <!-- エラーメッセージ終了 -->
~~~

`view/edit.php`

~~~php+HTML
(略)

<body>
    <h1>たけしのページの掲示板</h1>
    <h2>記事編集</h2>
    
    <!-- 以下を追記 -->
    
    <!-- エラーメッセージ表示エリア -->
    <?php if (isset($errmsg)) :?>
    <div class="errormsg">
        <p><?php echo htmlentities($errmsg, ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>
        </p>
    </div>
    <?php endif; ?>
    <!-- エラーメッセージ終了 -->
    
    <!-- ここまで -->
    
    <!-- 記事入力エリア -->
    <div class="input_area">
        
(略)
~~~

以上。