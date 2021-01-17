# 記事削除機能のテストを書く

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## テスト

### `DeleteDBPostData`

`testUpdateDBPostData`の最後で後片付けとして記事データを削除していたのを、`testDeleteDBPostData`に引き継ぐようにした。

~~~php
    // 8. 後片付け
    $sql = "delete from posts where id = $wrongData[id]";
    $smt = self::$pdo->query($sql);
}

↓
    
    // 8. testDeleteDBPostDataへ引き継ぐ
    return $data;
}
~~~

そうした後に以下のようにテストを書く。

`tests/GetFormActionTest.php`

~~~php
/**
 * @depends testUpdateDBPostData
 */
public function testDeleteDBPostData($originalPostData)
{
    // 1. GetFormActionインスタンスを生成
    $action = new GetFormAction();

    // 2. パスワードが違うと削除できない
    $result = $action->DeleteDBPostData((int)$originalPostData['id'], "hugahuga");
    $this->assertFalse($result);

    // 3. SQL文で直接記事を取得を試みる
    $sql = "select * from posts where id = $originalPostData[id]";
    $stmt = self::$pdo->query($sql);
    $actual_fetch = $stmt->fetch();

    // 4. 論理削除されていないことを確認
    $this->assertNull($actual_fetch['deleted_at']);

    // 5. パスワードが正しいと削除できる
    $result = $action->DeleteDBPostData((int)$originalPostData['id'], $originalPostData['password']);
    $this->assertTrue($result);

    // 6. SQL文で直接記事を取得を試みる
    $sql = "select * from posts where id = $originalPostData[id]";
    $stmt = self::$pdo->query($sql);
    $actual_fetch = $stmt->fetch();

    // 7. 論理削除されていることを確認
    $this->assertNotNull($actual_fetch['deleted_at']);

    // 8. GetDBPostDataで記事が取得できないことを確認
    $actual_results = $action->GetDBPostData();
    $postDatafound = false;
    foreach ($actual_results as $actual_result) {
        if ($actual_result['id'] == $originalPostData['id']) {
            $postDatafound = true;
        }
    }
    $this->assertFalse($postDatafound);

    // 9. 後片付け
    $sql = "delete from posts where id = $originalPostData[id]";
    $smt = self::$pdo->query($sql);
}
~~~

今は実装していないのでエラーが出て終わる。

~~~shell
> ./phpunit tests/
PHPUnit 9.0.0 by Sebastian Bergmann and contributors.

...............E                                                  16 / 16 (100%)

Time: 00:00.598, Memory: 4.00 MB

There was 1 error:

1) GetFormActionTest::testDeleteDBPostData
Error: Call to undefined method GetFormAction::DeleteDBPostData()

D:\work\HTML\raspberrypi-server\test\html\bbs\tests\GetFormActionTest.php:322

ERRORS!
Tests: 16, Assertions: 50, Errors: 1.
~~~

次は実装へ。