# 記事削除機能

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## `DeleteDBPostData`

`model/GetFormAction.php`に以下を追記。

~~~php
public function DeleteDBPostData(int $postId, string $password){
    // パスワードを確認
    $target_data = $this->GetDBOnePostData($postId);
    if ($password != $target_data['password']) {
        return false;
    }

    $smt = $this->pdo->prepare('update posts set deleted_at=now() where id=:id');
    $smt->bindParam(':id', $postId, PDO::PARAM_INT);
    return $smt->execute();
}
~~~

さらに`GetDBPostData`のSQL文を以下のように変更。

~~~
'select * from posts order by posted_at DESC'

↓

'select * from posts where deleted_at is null order by posted_at DESC'
~~~

ということは`GetDBOnePostData`も同じように削除された記事は取得できたらいけないのでSQL文を変更。

~~~
'select * from posts where id = :id'

↓

'select * from posts where id = :id and deleted_at is null'
~~~

テストにもそのことを確認する処理を書いておく。

`tests/GetFormActionTest.php`

~~~php
    // 8. GetDBPostDataで記事が取得できないことを確認
    $actual_results = $action->GetDBPostData();
    $postDatafound = false;
    foreach ($actual_results as $actual_result) {
        if ($actual_result['id'] == $originalPostData['id']) {
            $postDatafound = true;
        }
    }
    $this->assertFalse($postDatafound);

    // 以下を追記

    // 9. GetDBOnePostDataでも記事が取得できないことを確認
    $getOneData_result = $action->GetDBOnePostData((int)$originalPostData['id']);
    $this->assertFalse($getOneData_result);

    // ここまで

    // 10. 後片付け
    $sql = "delete from posts where id = $originalPostData[id]";
    $smt = self::$pdo->query($sql);
}
~~~

さあテストしてみる。

~~~shell
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

................                                                  16 / 16 (100%)

Time: 00:00.639, Memory: 4.00 MB

OK (16 tests, 56 assertions)
~~~

あっ、通った。

正直あっさり通られるとそれはそれで不安になる。とりあえずViewに組み込んで実際に試してみよう。