# コード見直し　～TODOのお掃除①～

コードを書きながら「あー、ここはもっとこうした方がいいけど、直すの後にしよ」っていうところに`TODO:`から始まるコメント行を書いて、後から見直しやすくするということはよくある話。今回はそれらを片付けていく。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 記事編集画面へ遷移するときのIDが不正だった場合、別ページを表示させる

該当箇所。

`index.php`

~~~php
switch ($eventId) {
    case 'save':
        $saveResult = $action->SaveDBPostData($_POST);
        require('./view/post.php');
        break;
    case 'update':
        $updateResult = $action->UpdateDBPostData($_POST);
        require('./view/post.php');
        break;
    case 'delete':
        $deleteResult = $action->DeleteDBPostData((int)$_POST['id'], $_POST['password']);
        require('./view/post.php');
        break;
    default:
        $params = $action->GetParam();
        switch ($params['mode']) {
            case 'edit':
                $edit_data = $action->GetDBOnePostData($params['id']);
                if (is_array($edit_data)) {
                    require('./view/edit.php');
                    break;
                }
                // TODO: 本当は不正なIDはエラーページを一旦表示して通常ページにリダイレクトするようにすべき
                // no break
            default:
                require('./view/post.php');
                break;
        }
        break;
}
~~~

こういうのはViewを作ってしまえば早い。以下のファイルを新規作成。

`view/id_error.php`

~~~php+HTML
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>たけしのページの掲示板</title>
    <meta http-equiv="Refresh" content="5; URL=../">
</head>

<body>
    <h1>たけしのページの掲示板</h1>
    <h2>不正なパラメーター</h2>
    <p>5秒後、BBSページに戻ります。</p>
    <p>自動で戻らない場合は<a href="../">こちら</a>。</p>
</body>

</html>
~~~

コントローラーを修正。

`index.php`

~~~php
switch ($params['mode']) {
    case 'edit':
        $edit_data = $action->GetDBOnePostData($params['id']);
        if (is_array($edit_data)) {
            require('./view/edit.php');
            break;
        }
        // no break
    default:
        require('./view/post.php');
        break;
}

↓
    
switch ($params['mode']) {
    case 'edit':
        $edit_data = $action->GetDBOnePostData($params['id']);
        if (is_array($edit_data)) {
            require('./view/edit.php');
        } else {
            require('./view/id_error.php');
        }
        break;
    default:
        require('./view/post.php');
        break;
}
~~~

また、`view/post.php`から以下を削除。

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

<!-- 以下を削除 -->
<?php elseif (isset($edit_data) && $edit_data == false) : ?>
<div class="errormsg">
    <p>無効なパラメーターが指定されました。</p>
</div>
<!-- ここまで　-->

<?php endif; ?>
<!-- エラーメッセージ終了 -->
~~~

## 投稿されたデータが正当なものかどうかのチェックを`===`を使ってチェック

`model/GetFormAction.php`の`SaveDBPostData`と`UpdateDBPostData`に該当箇所がある。

~~~php
public function SaveDBPostData($data)
    {
        // 渡されたデータが正当なものかどうか
        // TODO: ===を使って型までチェック
        if (($data['name'] == '') or ($data['email'] == '') or ($data['body'] == '') or ($data['password'] == '')) {
            return false;
        }
    if (((mb_strlen($data['name'])) > 100) or ((mb_strlen($data['email']) > 256)) or (mb_strlen($data['body']) > 5000) or (mb_strlen($data['password']) > 50) or (mb_strlen($data['password']) < 4)) {
        return false;
    }
    (略)

public function UpdateDBPostData(array $data)
{
    // 渡されたデータが正当なものかどうか
    // TODO: SaveDBPostDataとコードがほとんど被ってる
    if (($data['id'] == '') or ($data['name'] == '') or ($data['email'] == '') or ($data['body'] == '') or ($data['password'] == '')) {
        return false;
    }
    if ((! ctype_digit($data['id'])) or ((mb_strlen($data['name'])) > 100) or ((mb_strlen($data['email']) > 256)) or (mb_strlen($data['body']) > 5000) or (mb_strlen($data['password']) > 50) or (mb_strlen($data['password']) < 4)) {
        return false;
    }
    (略)
~~~

`==`を`===`に変えて、型まで一致してるかどうかのチェックは簡単だけど、コードが被っているのもついでに何とかしたい。

ということでまとめるメソッドを作ってしまう。

~~~php
private function IsDataIncorrect($data)
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
~~~

そして、`SaveDBPostData`と`UpdateDBPostData`を書き換える。

~~~php
public function SaveDBPostData($data)
{
    // 渡されたデータが正当なものかどうか
    if ($this->IsDataIncorrect($data)) {
        return false;
    }
    (略)
    
public function UpdateDBPostData(array $data)
{
    // 渡されたデータが正当なものかどうか
    if ($this->IsDataIncorrect($data)) {
        return false;
    }
    (略)
~~~

テストも通った。

~~~shell
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

................                                                  16 / 16 (100%)

Time: 00:00.900, Memory: 4.00 MB

OK (16 tests, 56 assertions)
~~~

## `DeleteDBPostData`で引数チェックができていない

~~~php
public function DeleteDBPostData(int $postId, string $password)
{
    // TODO: 引数のバリデーションが出来ていない

    // パスワードを確認
    // TODO: UpdateDBPostDataとコードが被っている
    $target_data = $this->GetDBOnePostData($postId);
    if ($password != $target_data['password']) {
        return false;
    }

    $smt = $this->pdo->prepare('update posts set deleted_at=now() where id=:id');
    $smt->bindParam(':id', $postId, PDO::PARAM_INT);
    return $smt->execute();
}
~~~

設計で適当に引数を`int`と`string`なんてしたもんだからそういうコーディングになってしまったけど、普通に`$_POST`を受け取って、内部で`['id']`と`['password']`だけ使えばいいんじゃないか？

修正後

~~~php
public function DeleteDBPostData($data)
{
    // 渡されたデータが正当なものかどうか
    if ($this->IsDataIncorrect($data)) {
        return false;
    }

    // パスワードを確認
    // TODO: UpdateDBPostDataとコードが被っている
    $target_data = $this->GetDBOnePostData((int)$data['id']);
    if ($data['password'] != $target_data['password']) {
        return false;
    }

    $smt = $this->pdo->prepare('update posts set deleted_at=now() where id=:id');
    $postId = (int)$data['id'];
    $smt->bindParam(':id', $postId, PDO::PARAM_INT);
    return $smt->execute();
}
~~~

`index.php`

~~~php
    case 'delete':
        $deleteResult = $action->DeleteDBPostData((int)$_POST['id'], $_POST['password']);
        require('./view/post.php');
        break;

↓
    
    case 'delete':
        $deleteResult = $action->DeleteDBPostData($_POST);
        require('./view/post.php');
        break;
~~~

`GetFormActionTest.php`

~~~php
// 2. パスワードが違うと削除できない
$result = $action->DeleteDBPostData((int)$originalPostData['id'], "hugahuga");
$this->assertFalse($result);

↓
    
// 2. パスワードが違うと削除できない
$wrongData = $originalPostData;
$wrongData['password'] = "hugahuga";
$result = $action->DeleteDBPostData($wrongData);
$this->assertFalse($result);

--------

// 5. パスワードが正しいと削除できる
$result = $action->DeleteDBPostData((int)$originalPostData['id'], $originalPostData['password']);
$this->assertTrue($result);

↓

// 5. パスワードが正しいと削除できる
$result = $action->DeleteDBPostData($originalPostData);
$this->assertTrue($result);
~~~

テストも通った。

~~~shell
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

................                                                  16 / 16 (100%)

Time: 00:00.678, Memory: 4.00 MB

OK (16 tests, 56 assertions)
~~~

