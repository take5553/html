# コード見直し　～セキュリティを考慮する①～　XSS対策

セキュリティは何が正解なのかよく分かっていないけど、とりあえずすぐできるやつから。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## XSS

現状ではXSS（クロスサイトスクリプティング）の脆弱性がある。

例えば

![image-20210122222740094](image/security/image-20210122222740094.png)

という内容にして送信をクリックすると、メッセージボックスが出せる。

![image-20210122222814953](image/security/image-20210122222814953.png)

メッセージボックスだけじゃなく全てのJavaScriptのコードが実行可能なので、悪意のあるコードを掲示板に書き込むだけでアクセスしてきた他のユーザーが全員そのコードを強制的に実行してしまうことになる。

このように簡単にJavaScriptが埋め込めてしまうので、現状ではセキュリティ的によろしくない。

## 対策

HTMLタグに使われるような文字`<`とか`>`とかをエスケープして表示をすればよい。

エスケープに使うのはPHP関数の`htmlentities`。大体表示の直前で使えば良いらしい。

参考：[PHP: htmlentities \- Manual](https://www.php.net/manual/ja/function.htmlentities.php)

`view/post.php`

~~~php+HTML
(略)

<!-- 記事表示エリア -->
<div class="posts">
    <?php if (!empty($post_data)) : ?>
    <div class="post">
        <?php foreach ($post_data as $post) :?>
        <div class="name">
            <p>名前：<a
                     href="mailto:<?php echo htmlentities($post['email'], ENT_HTML5 | ENT_QUOTES, "UTF-8"); ?>"><?php echo htmlentities($post['name'], ENT_HTML5 | ENT_QUOTES, "UTF-8"); ?></a>
                <a
                   href="edit/<?php echo htmlentities($post['id'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">編集・削除</a>
            </p>
        </div>
        <div class="post_body">
            <p><?php echo htmlentities($post['body'], ENT_HTML5 | ENT_QUOTES, "UTF-8"); ?>
            </p>
        </div>
        <div class="posted_at">
            <p>投稿日時：<?php echo htmlentities($post['posted_at'], ENT_HTML5 | ENT_QUOTES, "UTF-8"); ?>
            </p>
        </div>
        <?php if ($post['posted_at'] != $post['updated_at']) : ?>
        <p>
            更新日時：<?php echo htmlentities($post['updated_at'], ENT_HTML5 | ENT_QUOTES, "UTF-8"); ?>
        </p>
        <?php endif;?>
        <?php endforeach; ?>
    </div>
    <?php endif; ?>
</div>
<!-- 記事表示エリア終了 -->

(略)
~~~

`view/edit.php`

~~~php+HTML
(略)

<!-- 記事入力エリア -->
<div class="input_area">
    <form action="../index.php" method="post" id="post_form">
        <p>
            名前：<br>
            <input type="text" name="name" id="name"
                   value="<?php echo htmlentities($edit_data['name'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
        </p>
        <p>
            メールアドレス：<br>
            <input type="email" name="email" id="email"
                   value="<?php echo htmlentities($edit_data['email'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
        </p>
        <p>
            本文：<br>
            <textarea name="body" id="body" cols="30"
                      rows="10"><?php echo htmlentities($edit_data['body'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?></textarea>
        </p>
        <p>
            パスワード：<br>
            <input type="password" name="password" id="password">
        </p>
        <p>
            <input type="hidden" name="id"
                   value="<?php echo htmlentities($edit_data['id'], ENT_HTML5 | ENT_QUOTES, "UTF-8") ?>">
            <button name="eventId" value="update">更新</button><br><br>
            <button name="eventId" value="delete">削除</button>
        </p>
    </form>
</div>
<!-- 記事入力エリア終了 -->

(略)
~~~

[続く。](security2.html)