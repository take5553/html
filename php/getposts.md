# 記事表示機能

今度はDBに保存しているデータを取得してHTMLに配置していく部分。先にテストを書こうと思ったけど、記事投稿機能よりさらに簡単なので先に実装をする。

以下のサイトを参考にする。

[【プログラミング構築】データベースにアクセスして投稿データを抽出しよう \| 株式会社LIG](https://liginc.co.jp/282262)

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモート
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - Nginx 1.14.2
  - PHP 7.3.19-1~deb10u1
  - MariaDB 10.3.23

## モデル作成

DBから記事を取得するメソッド`GetDBPostData`を`model/GetFormAction.php`に追記する。

~~~php
public function GetDBPostData(){
    $stm = $this->pdo->prepare('select * from posts order by posted_at DESC');
    $stm->execute();
    $result = $stm->fetchAll(PDO::FETCH_ASSOC);

    return $result;
}
~~~

これで`$result`に2次元配列で記事データが入ることになる。

## ビューを作成

`view/post.php`の先頭、`<!DOCTYPE html>`よりも前に以下を追記。

~~~php+HTML
<?php
$post_data = $action->GetDBPostData();
?>
~~~

これでHTMLファイル上で、DBから取ってきた記事データを扱うことができる。

記事表示エリアも以下のように変更。

~~~php+HTML
    <!-- 記事表示エリア -->
    <div class="posts">
        <?php if (!empty($post_data)) : ?>
        <div class="post">
            <?php foreach ($post_data as $post) :?>
            <div class="name">
                <p>名前：<a
                        href="mailto:<?php echo $post['email']; ?>"><?php echo $post['name']; ?></a>
                </p>
            </div>
            <div class="post_body">
                <p><?php echo $post['body']; ?>
                </p>
            </div>
            <div class="posted_at">
                <p><?php echo $post['posted_at']; ?>
                </p>
            </div>
            <?php if ($post['posted_at'] != $post['updated_at']) : ?>
            <p>
                <?php echo $post['updated_at']; ?>
            </p>
            <?php endif;?>
            <?php endforeach; ?>
        </div>
        <?php endif; ?>
    </div>
    <!-- 記事表示エリア終了 -->
~~~

保存終了し、XAMPPが動いていることを確認して、ブラウザで`localhost`にアクセス。

![image-20201228100115147](image/getposts/rs-image-20201228100115147.png)

相変わらず見た目がひどい。でも一応記事データが取得できている。

試しにこのページ上から記事を投稿してみる。

![image-20201228100343663](image/getposts/image-20201228100343663.png)

送信ボタンを押すと、スーパー地味だけど一応投稿できている。

![image-20201228100413933](image/getposts/rs-image-20201228100413933.png)