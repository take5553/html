# 記事表示のテストを書く

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

## テスト内容

* 対象クラス：`GetFormAction`
* 対象メソッド：`GetDBPostData`
* 引数：なし
* 戻り値：2次元配列

### 正常系

1. 直接SQL文で記事をDBに保存
2. `GetDBPostData`を実行
3. 先ほど保存した記事データがちゃんと読み込めているか評価
4. 後片付け

### 準正常系・異常系

なんかある？DBから取ってこれなかったってことはDBに接続できなかったか記事が登録されていないかぐらいしか思いつかん。これは引数が無いからかな。引数が必要になってきたらまた考えよう。

## テストを書く

以下を`tests/GetFormActionTest.php`に追記。

~~~php
/**
 * @dataProvider getDataProvider
 */
public function testGetDBPostData($data)
{
    // 1. GetFormActionインスタンスを生成
    $action = new GetFormAction();

    // 2. SQL文で直接記事を登録
    try {
        $pdo = new PDO(PDO_DSN, DATABASE_USER, DATABASE_PASSWORD);
        $pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        $pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    } catch (PDOException $e) {
        print('Error:'.$e->getMessage());
        die();
    }
    $smt = $pdo->prepare('insert into posts (name,email,body,password,posted_at,updated_at) values(:name,:email,:body,:password,now(),now())');
    $smt->bindParam(':name', $data['name'], PDO::PARAM_STR);
    $smt->bindParam(':email', $data['email'], PDO::PARAM_STR);
    $smt->bindParam(':body', $data['post_body'], PDO::PARAM_STR);
    $smt->bindParam(':password', $data['password'], PDO::PARAM_STR);
    $smt->execute();

    // 3. GetDBPostDataで記事データ取得
    $actual_result = $action->GetDBPostData();
    $testPost = $actual_result[0];
    
    // 4. それぞれ確認
    $this->assertEquals($data['name'], $testPost['name']);
    $this->assertEquals($data['email'], $testPost['email']);
    $this->assertEquals($data['post_body'], $testPost['body']);
    $this->assertEquals($data['password'], $testPost['password']);

    // 5. 後片付け

    $sql = "delete from posts where name = '$data[name]'";
    $smt = $pdo->query($sql);
    $pdo = null;
}

public function getDataProvider()
{
    return [
        'successful' => [[
            'name' => 'gettest',
            'email' => 'get@get.get',
            'post_body' => 'gettest',
            'password' => 'getget'
        ]]
    ];
}
~~~

今回もデータプロバイダを作ることにした。それに伴い、こっそり前回の`testSaveDBPostData`のデータプロバイダである`postDataProvider`の名前も`saveDataProvider`に変更。

~~~php
/**
 * @dataProvider saveDataProvider
 */
public function testSaveDBPostData($data, $expected)
{
    （略）
}

public function saveDataProvider()
{
    （略）
}
~~~

テストを実行。

~~~shell
> ./phpunit tests/GetFormActionTest.php
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

.......                                                             7 / 7 (100%)

Time: 00:00.549, Memory: 4.00 MB

OK (7 tests, 20 assertions)
~~~

いいねいいね。

## テストコード見直し

ここでは前から気になっていた「テストごとにPDOインスタンスを生成する」という部分を見直す。

`testSaveDBPostData`と`testGetDBPostData`で同じことをしているので、これを

* テスト全体が始まる前にDBに接続
* テスト全体が終わった後にDBの接続を解除

とする。

テスト全体が始まる前や終わった後に何かする場合は、`setUpBeforeClass`と`tearDownAfterClass`を作ってそこに書く。

例えば

> ~~~php
> <?php
> use PHPUnit\Framework\TestCase;
> 
> class DatabaseTest extends TestCase
> {
>     protected static $dbh;
> 
>     public static function setUpBeforeClass(): void
>     {
>         self::$dbh = new PDO('sqlite::memory:');
>     }
> 
>     public static function tearDownAfterClass(): void
>     {
>         self::$dbh = null;
>     }
> }
> ~~~
>
> [4\. フィクスチャ — PHPUnit latest Manual](https://phpunit.readthedocs.io/ja/latest/fixtures.html#fixtures-sharing-fixture)

こんな感じ。クラスプロパティにしないといけないのがよく分からんけど、その通りにしておく。

あまりこのフィクスチャ（テスト前の準備とか後片付けとか）の共有化はしない方がいいみたい。

> このようにフィクスチャを共有することがテストの価値を下げてしまうということを、 まだうまく伝え切れていないかもしれません。問題なのは、 各オブジェクトが疎結合になっていないという設計なのです。 複数が連携しているようなテストを作って設計上の問題から目をそらしてしまうのではなく、 きちんと設計しなおした上で、スタブ ([テストダブル](https://phpunit.readthedocs.io/ja/latest/test-doubles.html#test-doubles) を参照ください) を使用するテストを書くことをお勧めします。
>
> [4\. フィクスチャ — PHPUnit latest Manual](https://phpunit.readthedocs.io/ja/latest/fixtures.html#fixtures-sharing-fixture)

なので、DB接続だけにしておく。

### フィクスチャ実装

`tests/GetFormActionTest.php`の先頭に以下を追記。

~~~php
protected static $pdo;

public static function setUpBeforeClass(): void
{
    try {
        self::$pdo = new PDO(PDO_DSN, DATABASE_USER, DATABASE_PASSWORD);
        self::$pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
        self::$pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
    } catch (PDOException $e) {
        print('Error:'.$e->getMessage());
        die();
    }
}

public static function tearDownAfterClass(): void
{
    self::$pdo = null;
}
~~~

あとは`testSaveDBPostData`と`testGetDBPostData`からPDOインスタンス生成処理を削除し、ところどころに出てくる`$pdo`を`self::$pdo`に書き換える。PDOインスタンス生成処理を削除したらVSCodeがマズイところを強調してくれるので、それを頼りに探していけば楽。

テスト実行。

~~~shell
> ./phpunit tests/GetFormActionTest.php
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

.......                                                             7 / 7 (100%)

Time: 00:00.573, Memory: 4.00 MB

OK (7 tests, 20 assertions)
~~~

ちゃんと動いてる。