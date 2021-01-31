# 記事データをファーストクラスコレクションとしてラップ

ファーストクラスコレクションとは

* 普通の配列や連想配列のような`変数名[キー名 or インデックス]`で扱えるようなものをラップするクラス

と言える。

大体こういう変数を作るということは、ループに使ったり追加・削除、あるいは全部で何個とかそういう処理が付いて回るけど、それをなるべくこのファーストクラスコレクション内に移譲してしまおうという使い方が多い。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 概要

参考サイトを見て勉強したところによると、今回の掲示板プログラムでは

* `Posts`クラス・・・`post`クラスを配列化したものを持つ
* `Post`クラス・・・記事1つ分のデータを持つ

とした方がよさそう。

## 実装

### `Post`クラス

`Post`クラスは1記事を担当するけど、処理の流れを考えると`Post`クラスの担当になりそうなデータは3種類ある。

* ユーザーが投稿した記事データ

  （渡されるデータは`name`、`email`、`body`、`password`）

* ユーザーが編集した記事データ

  （渡されるデータは`id`、`name`、`email`、`body`、`password`）

* DBから取得した個々の記事データ

  （渡されるデータは`id`、`name`、`email`、`body`、`password`、`posted_at`、`updated_at`）

ここでは「DBから取得した個々の記事データ」をクラス化することにする。

`model/post.php`を新規作成し以下を記述。

~~~php
<?php
require_once('./lib/func.php');

class Post
{
    private $id;
    private $name;
    private $email;
    private $body;
    private $password;
    private $posted_at;
    private $updated_at;

    public function __construct($postArray)
    {
        $this->id = $postArray['id'];
        $this->name = $postArray['name'];
        $this->email = $postArray['email'];
        $this->body = $postArray['body'];
        $this->password = $postArray['password'];
        $this->posted_at = $postArray['posted_at'];
        $this->updated_at = $postArray['updated_at'];
    }

    public function TheId()
    {
        return h($this->id);
    }

    public function TheName()
    {
        return h($this->name);
    }

    public function TheEmail()
    {
        return h($this->email);
    }

    public function TheBody()
    {
        return h($this->body);
    }

    public function ThePostedDate()
    {
        return h($this->posted_at);
    }

    public function TheUpdatedDate()
    {
        return $this->IsUpdated() ? h($this->updated_at) : '';
    }

    public function IsUpdated()
    {
        return $this->posted_at === $this->updated_at;
    }
}
~~~

また、`model/Posts.php`を作成し、以下を記述。

~~~php
<?php
require_once('../lib/func.php');
    
class Posts implements IteratorAggregate
{
    private $posts;

    public function __construct($postsFromDB)
    {
        // $postsFromDBの中身は
        // 1. false（記事数0）
        // 2. 一次元配列（記事数1）
        // 3. 二次元配列（記事数2以上）
        if (! is_array($postsFromDB)) {
            $this->posts = array();
        } elseif (array_depth($postsFromDB) === 1) {
            $this->posts = array(new Post($postsFromDB));
        } else {
            $this->posts = array();
            foreach ($postsFromDB as $post) {
                $this->Add(new Post($post));
            }
        }
    }

    public function Add(Post $post)
    {
        $this->posts[] = $post;
    }

    public function HavePosts()
    {
        return count($this->posts) !== 0;
    }

    public function getIterator()
    {
        return new ArrayIterator($this->posts);
    }
}
~~~

コンストラクタ―で出てくる`array_depth`は[【php】配列の深さ（次元）を調べる方法 at softelメモ](https://www.softel.co.jp/blogs/tech/archives/58)からコピペした。

`lib/func.php`の末尾に以下を追記。何気にDocBlocker使うの初めて。

~~~php
/**
 * https://www.softel.co.jp/blogs/tech/archives/58
 * ↑ここからコピー
 *
 * @param array $a
 * @param integer $c
 * @return int
 */
function array_depth($a, $c = 0)
{
    if (is_array($a) && count($a)) {
        ++$c;
        $_c = array($c);
        foreach ($a as $v) {
            if (is_array($v) && count($v)) {
                $_c[] = array_depth($v, $c);
            }
        }
        return max($_c);
    }
    return $c;
}

~~~

続いて、`model/GetFormAction.php`でこれらを読み込み、

~~~php
<?php
require_once('Validator.php');
require_once('Post.php'); // ←追記
require_once('Posts.php'); // ←追記
class GetFormAction
{
    private $pdo;
    private $validator;
    
    (略)
~~~

`GetDBPostData`を以下のように改造する。

~~~php
public function GetDBPostData()
{
    $stm = $this->pdo->prepare('select * from posts where deleted_at is null order by posted_at DESC');
    $stm->execute();

    $results = $stm->fetchAll(PDO::FETCH_ASSOC);

    return new Posts($results); // ←変更
}
~~~

そして最後に`view/post.php`の記事表示部分を以下のように変更。

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
                <p><?php echo nl2br($post->TheBody()); ?>
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
</body>

</html>
~~~

これで今までと同じように動いた。

## テストを改造

絶対動かないと思うけど一応テストをしてみる。

~~~shell
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

......E........E                                                  16 / 16 (100%)

Time: 00:01.207, Memory: 4.00 MB

There were 2 errors:

1) GetFormActionTest::testGetDBPostData with data set "successful" (array('gettest', 'get@get.get', 'gettest', 'getget'))
Error: Cannot use object of type Posts as array

D:\work\HTML\raspberrypi-server\test\html\bbs\tests\GetFormActionTest.php:135

2) GetFormActionTest::testDeleteDBPostData
Error: Cannot use object of type Post as array

D:\work\HTML\raspberrypi-server\test\html\bbs\tests\GetFormActionTest.php:352

ERRORS!
Tests: 16, Assertions: 50, Errors: 2.
~~~

えっ？2つだけ？

と思ってよく見たら`GetFormActionTest.php`で`GetDBPostData`が使われている箇所だけエラーが出てるだけで、`GetDBPostData`しか変えてないんだからそりゃそうだ。

修正する。

`tests/GetFormActionTest.php`

~~~php
public function testGetDBPostData($data)
{
    // 1. GetFormActionインスタンスを生成
    $action = new GetFormAction();

    // 2. SQL文で直接記事を登録
    $hashed_password = password_hash($data['password'], PASSWORD_DEFAULT);
    $smt = self::$pdo->prepare('insert into posts (name,email,body,password,posted_at,updated_at) values(:name,:email,:body,:password,now(),now())');
    $smt->bindParam(':name', $data['name'], PDO::PARAM_STR);
    $smt->bindParam(':email', $data['email'], PDO::PARAM_STR);
    $smt->bindParam(':body', $data['body'], PDO::PARAM_STR);
    $smt->bindParam(':password', $hashed_password, PDO::PARAM_STR);
    $smt->execute();

    // 3. GetDBPostDataで記事データ取得
    $actual_result = $action->GetDBPostData();
    
// 以下を変更
    
    foreach ($actual_result as $testPost) {
        break;
    }

    // 4. それぞれ確認
    $this->assertEquals(h($data['name']), $testPost->TheName());
    $this->assertEquals(h($data['email']), $testPost->TheEmail());
    $this->assertEquals(h($data['body']), $testPost->TheBody());
    //$this->assertTrue(password_verify($data['password'], $testPost['password']));
    
// ここまで

    // 5. 後片付け
    $sql = "delete from posts where name = '$data[name]'";
    $smt = self::$pdo->query($sql);
}
~~~

`foreach`で一つだけ取り出してすぐ`break`することで1つだけ取り出すようにする。後パスワードを確認する仕組みはまだ実装していないのでとりあえずテストからは省く。

もう一つ。

~~~php
public function testDeleteDBPostData($originalPostData)
{
    (略)

    // 8. GetDBPostDataで記事が取得できないことを確認
    $actual_results = $action->GetDBPostData();
    $postDatafound = false;
    foreach ($actual_results as $actual_result) {
        if ($actual_result->TheId() == h($originalPostData['id'])) { // ←判断条件変更
            $postDatafound = true;
        }
    }
    $this->assertFalse($postDatafound);

    (略)
    
}
~~~

これでテストが通る。

## 参考

[配列の処理をファーストクラスコレクションに組み替えてみる \- Qiita](https://qiita.com/harunbu/items/0ab2b2e179ae63e5aa8a)
[C\#で学ぶデザインパターン入門 ①Iterator \- Qiita](https://qiita.com/toshi0607/items/cdc589c58f21c0fc513d)
[PHPオブジェクト指向入門（前半） \- Qiita](https://qiita.com/mpyw/items/41230bec5c02142ae691)