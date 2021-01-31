# `GetFormAction`クラスのメソッドを整理

いよいよオブジェクト指向の観点からコードを整理していく。とりあえずの準備として`GetFormAction`クラスの中にあるメソッドのうち、実は`GetFormAction`内にある必要は無いものを洗い出す。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 概要

具体的には

* `GetParam()`
* `IsDataIncorrect($data)`

がいらないんじゃないかと思ってる。

というのも、この2つは`GetFormAction`が持つインスタンス変数（現状では`$pdo`のみ）を一切使用していないから。

なので、

* `GetParam()`→グローバル関数化
* `IsDataIncorrect($data)`→バリデーションを担当するクラスを作成し、そこに入れる

ということをする。

## `GetParam()`の移行

これは`lib/func.php`に移す。

~~~php
<?php

function h($str)
{
    return htmlentities($str, ENT_HTML5 | ENT_QUOTES, "UTF-8");
}

function GetParam()
{
    $ret = array(
        'mode' => '',
        'id' => ''
    );
    $params = explode('/', $_SERVER['REQUEST_URI'], 5);
    if ($params[1] != 'bbs') {
        return $ret;
    }
    if ($params[2] != 'edit') {
        return $ret;
    }
    if (! ctype_digit($params[3])) {
        return $ret;
    }

    $ret['mode'] = $params[2];
    $ret['id'] = (int)$params[3];

    return $ret;
}
~~~

それに伴い

* `index.php`
* `GetFormAction.php`
* `GetFormActionTest.php`

に出てきている`$action->GetParam()`を`GetParam()`に修正していく。VSCodeが場所を教えてくれるはず。

そしてさらにそれに伴って`GetFormActionTest->testGetParam`で`GetFormAction`インスタンスを生成する必要がなくなったので削除。

~~~php
public function testGetParam($data, $expected)
{
    // 以下を削除
    // 1. GetFormActionインスタンスを生成
    $action = new GetFormAction();
    // ここまで

    // 2. リクエストURIセット
    $_SERVER['REQUEST_URI'] = $data['uri'];

    // 3. GetParamでパラメーター取得
    $actual_result = GetParam();

    // 4. 評価
    $this->assertEquals($expected['mode'], $actual_result['mode']);
    $this->assertEquals($expected['id'], $actual_result['id']);
}
~~~

また、テストモジュールに`func.php`を読み込ませることも忘れずに。

`tests/GetFormActionTest.php`

~~~php
<?php

use phpDocumentor\Reflection\Types\Boolean;
use PHPUnit\Framework\TestCase;

require('config/properties_for_test.php');
require('model/GetFormAction.php');
require_once('lib/func.php'); // ←追記

class GetFormActionTest extends TestCase
{
    (略)
~~~

## `IsDataIncorrect($data)`の移行

まず`model`フォルダの中に`Validator.php`というファイルを新しく作ってそこに移行する。

~~~php
<?php
class Validator
{
    public function IsDataIncorrect($data)
    {
        // idのチェック
        if (isset($data['id'])) {
            if ($data['id'] === '' or (! ctype_digit($data['id']))) {
                return true;
            }
        }

        // nameのチェック
        if ($data['name'] === '' or ((mb_strlen($data['name'])) > 100)) {
            return true;
        }

        // emailのチェック
        if ($data['email'] === '' or ((mb_strlen($data['email']) > 256))) {
            return true;
        }

        // bodyのチェック
        if ($data['body'] === '' or (mb_strlen($data['body']) > 5000)) {
            return true;
        }

        // passwordのチェック
        if ($data['password'] === '' or (mb_strlen($data['password']) > 50) or (mb_strlen($data['password']) < 4)) {
            return true;
        }

        return false;
    }
}
~~~

`model/GetFormAction.php`の先頭で読み込みを行い、コンストラクタ―で`Validator`インスタンスを生成し、インスタンス変数に格納。

~~~php
<?php
require_once('Validator.php'); // ←追記
class GetFormAction
{
    private $pdo;
    private $validator; // ←追記

    public function __construct()
    {
        $this->validator = new Validator(); // ←追記

        try {
            $this->pdo = new PDO(PDO_DSN, DATABASE_USER, DATABASE_PASSWORD);
            $this->pdo->setAttribute(PDO::ATTR_ERRMODE, PDO::ERRMODE_EXCEPTION);
            $this->pdo->setAttribute(PDO::ATTR_EMULATE_PREPARES, false);
        } catch (PDOException $e) {
            header('Content-Type: text/plain; charset=UTF-8', true, 500);
            exit($e->getMessage());
        }
    }
    
    (略)
~~~

そして、`SaveDBPostData`、`UpdateDBPostData`、`DeleteDBPostData`の各メソッドで`$data`をチェックしているところを変える。

~~~php
// 渡されたデータが正当なものかどうか
if ($this->IsDataIncorrect($data)) {
    return false;
}

↓

// 渡されたデータが正当なものかどうか
if ($this->validator->IsDataIncorrect($data)) {
    return false;
}
~~~

ついでに各メソッドのローカル変数の名前を`$data`から`$postData`に変えておく。（短すぎる名前は逆に検索に引っかかりすぎて鬱陶しい）

~~~php
public function SaveDBPostData($postData) // ←引数名を変える(以下全部)
{
    // 渡されたデータが正当なものかどうか
    if ($this->validator->IsDataIncorrect($postData)) {
        return false;
    }

    // パスワードのハッシュ化
    $postData['password'] = password_hash($postData['password'], PASSWORD_DEFAULT);
    if ($postData['password'] === false) {
        return false;
    }

    // 投稿された記事をDBに保存
    $smt = $this->pdo->prepare('insert into posts (name,email,body,password,posted_at,updated_at) values(:name,:email,:body,:password,now(),now())');
    $smt->bindParam(':name', $postData['name'], PDO::PARAM_STR);
    $smt->bindParam(':email', $postData['email'], PDO::PARAM_STR);
    $smt->bindParam(':body', $postData['body'], PDO::PARAM_STR);
    $smt->bindParam(':password', $postData['password'], PDO::PARAM_STR);
    return $smt->execute();
}
~~~

他のメソッドも同様に。