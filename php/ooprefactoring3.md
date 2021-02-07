# `GetDBOnePostData`の戻り値を`Post`インスタンスに変更

前回が`GetDBPostData`の変更だったので、今回は記事1つだけ取り出す`GetDBOnePostData`を変更する。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 実装

`model/GetFormAction.php`

~~~php
(略)

public function GetDBOnePostData(int $postId)
{
    $stm = $this->pdo->prepare('select * from posts where id = :id and deleted_at is null');
    $stm->bindParam(':id', $postId, PDO::PARAM_INT);
    $stm->execute();
    $result = $stm->fetch(PDO::FETCH_ASSOC);

    return is_array($result) ? new Post($result) : false; // ←ここだけ修正
}

(略)
~~~

`index.php`

セッション切れの救済措置を`view/edit.php`に移した。

理由は`Post`インスタンスはDBから引き出したデータなので、基本的に編集を受け付けるようには作るつもりがないから。

~~~php
(略)

	default:
        switch ($params['mode']) {
            case 'edit':
                $edit_data = $action->GetDBOnePostData($params['id']);
                if (is_object($edit_data)) { // ←条件を変更

                    if ($reupdateFlag) {
                        
                        // $errmsg以外の処理を削除
                        $errmsg = $_SESSION['errmsg'];
                        
                    }
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

`view/edit.php`

`<input>`や`<textarea>`にデータを配置する処理を変更。

ここで本文を出力するときに`htmlentities`と`preg_replace`の関係がややこしくなってしまったので、`Post`の`TheBody()`の出力は`htmlentites`を通さないことにした。

~~~php+HTML
(略)

    <!-- 記事入力エリア -->
    <div class="input_area">
        <form action="../index.php" method="post" id="post_form">
            <p>
                名前：<br>
                <input type="text" name="name" id="name"
                    value="<?php echo $reupdateFlag ? h($_SESSION['name']) : $edit_data->TheName() ;?>">
            </p>
            <p>
                メールアドレス：<br>
                <input type="email" name="email" id="email"
                    value="<?php echo $reupdateFlag ? h($_SESSION['email']) : $edit_data->TheEmail() ?>">
            </p>
            <p>
                本文：<br>
                <textarea name="body" id="body" cols="30"
                    rows="10"><?php echo h(preg_replace("/(\r\n|\n|\r)/", "\n", $reupdateFlag ? $_SESSION['body'] : $edit_data->TheBody())) ?></textarea>
            </p>
            <p>
                パスワード：<br>
                <input type="password" name="password" id="password">
            </p>
            <p>
                <input type="hidden" name="id"
                    value="<?php echo h($edit_data->TheId()) ?>">
                <input type="hidden" name="token"
                    value="<?php echo h(password_hash(session_id(), PASSWORD_DEFAULT)) ?>">
                <button name="eventId" value="update">更新</button><br><br>
                <button name="eventId" value="delete">削除</button>
            </p>
        </form>
    </div>
    <!-- 記事入力エリア終了 -->
</body>
~~~

`model/Post.php`

~~~php
(略)

public function TheBody()
{
    return $this->body; // ←h()を取った
}

(略)
~~~

そうすると`view/post.php`で記事を表示させるときに`h()`を通す必要が出てきた。

~~~php+HTML
(略)

    <!-- 記事表示エリア -->
    <div class="posts">
        <?php if ($posts->HavePosts()) : ?>
        <div class="post">
            <?php foreach ($posts as $post) :?>
            <div class="name">
                <p>名前：<a
                        href="mailto:<?php echo $post->TheEmail(); ?>"><?php echo $post->TheName(); ?></a>
                    <a
                        href="edit/<?php echo $post->TheId(); ?>">編集・削除</a>
                </p>
            </div>
            <div class="post_body">
                <p><?php echo nl2br(h($post->TheBody())); ?> <!-- ←ここだけ修正 -->
                </p>
            </div>
            <div class="posted_at">
                <p>投稿日時：<?php echo $post->ThePostedDate(); ?>
                </p>
            </div>
            <?php if ($post->IsUpdated()) : ?>
            <p>
                更新日時：<?php echo $post->TheUpdatedDate(); ?>
            </p>
            <?php endif;?>
            <?php endforeach; ?>
        </div>
        <?php endif; ?>
    </div>
    <!-- 記事表示エリア終了 -->

(略)
~~~

また、記事編集や記事削除でパスワードを確認するときにエラーが出てしまうので、

* `Post`クラスにパスワードを確認するメソッドを追加
* `UpdateDBPostData`、`DeleteDBPostData`のパスワード確認処理を変更

`model/Post.php`

以下を末尾に追記。

~~~php
/**
 * パスワードを確認する
 *
 * @param string $inputPassword
 * @return bool
 */
public function password_verify($inputPassword)
{
    return password_verify($inputPassword, $this->password);
}
~~~

`model/GetFormAction.php`

パスワード確認処理を変更

~~~php
public function UpdateDBPostData(array $postData)
{
    (略)

    // パスワードを確認
    $old_data = $this->GetDBOnePostData((int)$postData['id']);
    if (! $old_data->password_verify($postData['password'])) { // ←ここを変更
        return false;
    }

    (略)
}

public function DeleteDBPostData($postData)
{
    (略)

    // パスワードを確認
    $old_data = $this->GetDBOnePostData((int)$postData['id']);
    // もし削除済みの記事を再度削除しようとしたとき（削除後トップページでリロードしてしまった時など）falseが返ってくる対策
    if ($old_data === false) {
        return false;
    }
    if (! $old_data->password_verify($postData['password'])) { // ←ここを変更
        return false;
    }

    (略)
}
~~~

予想通りめんどくさい変更になってしまった。

あまり納得のいっていない点は以下。

* セッション切れの救済措置

  今回はDBからViewへ流す用のクラスに焦点を当てたので、ユーザー入力を受け取るクラスを作るときに考える。

* `htmlentities`の処理

  本文の出力先がHTMLとtextareaの2種類あるのが原因。その判断を`Post`クラスにさせるのはちょっと酷かなー。

## テストを修正

`tests/GetFormActionTest.php`

パスワードが確認できるようになったので、`testGetDBPostData`でコメントアウトしていた部分を修正する。

~~~php
public function testGetDBPostData($data)
{

    (略)

    // 4. それぞれ確認
    $this->assertEquals(h($data['name']), $testPost->TheName());
    $this->assertEquals(h($data['email']), $testPost->TheEmail());
    $this->assertEquals(h($data['body']), $testPost->TheBody());
    $this->assertTrue($testPost->password_verify($data['password'])); // ←ここを修正

    (略)
    
}
~~~

また、`testGetDBOnePostData`はアサーション部分を全部変える。

~~~php
public function testGetDBOnePostData()
{

    (略)

    // 6. SQL文で取得したデータと比較
    // 全部変更
    $this->assertEquals(h($actual_fetch['name']), $actual_result->TheName());
    $this->assertEquals(h($actual_fetch['email']), $actual_result->TheEmail());
    $this->assertEquals($actual_fetch['body'], $actual_result->TheBody());
    $this->assertTrue($actual_result->password_verify($data['password']));

    (略)
    
}
~~~

