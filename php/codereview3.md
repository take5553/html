# コード見直し　～グローバル変数の整理～

適当に変数を増やしてしまったから見直す。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモートにはアップしない

## 概要

`index.php`の最初の方で変数の準備はしているけど、徹底しているわけではない。`filter_input`の使い方も知ったし、この際バリデーションも含めてちゃんと見直す。

基本的にはVSCodeの検索機能で一括置き換え。

## 整理

### `index.php`

| 変数名                | 用途                                     | 変更後              |
| --------------------- | ---------------------------------------- | ------------------- |
| `$action`             | `GetFormAction`インスタンス格納          |                     |
| `$eventId`            | 記事投稿・更新・削除の見分け             |                     |
| `$repost_flag`        | セッション切れで再投稿が必要な時に立つ   | `$repostFlag`       |
| `$reupdate_flag`      | セッション切れで再編集が必要な時に立つ   | `$reupdateFlag`     |
| `$filtered_post_data` | `$_POST`から受け取ったデータを格納       | `$postDataFromUser` |
| `$token`              | CSRF対策用トークン                       |                     |
| `$params`             | 記事編集画面を飛ぶときのパラメーター格納 |                     |
| `$errmsg`             | 処理に失敗したときのエラーメッセージ     |                     |
| `$url`                | （一時的）                               |                     |
| `$saveResult`         | （一時的）                               |                     |
| `$updateResult`       | （一時的）                               |                     |
| `$deleteResult`       | （一時的）                               |                     |
| `$edit_data`          | 記事編集画面へ渡すデータ（1記事のみ）    | `$editData`         |

### `view/post.php`

| 変数名       | 用途                                 | 変更後   |
| ------------ | ------------------------------------ | -------- |
| `$post_data` | 記事表示画面へ渡すデータ（複数記事） | `$posts` |
| `$post`      | `foreach`ループ内専用                |          |

あんまり置き換えるもの無かったな。

## 修正

大した修正ではないけど、`SaveDBPostData`、`UpdateDBPostData`および`DeleteDBPostData`に渡していた変数を`$_POST`から`$postDataFromUser`に変更。

`index.php`

~~~php
(略)

switch ($eventId) {
    case 'save':
        $saveResult = $action->SaveDBPostData($postDataFromUser);
        if ($saveResult == false) {
            $errmsg = "記事投稿に失敗しました。";
        }
        require('./view/post.php');
        break;
    case 'update':
        $updateResult = $action->UpdateDBPostData($postDataFromUser);
        if ($updateResult == false) {
            $errmsg = "記事編集に失敗しました。";
        }
        require('./view/post.php');
        break;
    case 'delete':
        $deleteResult = $action->DeleteDBPostData($postDataFromUser);
        if ($deleteResult == false) {
            $errmsg = "記事削除に失敗しました。";
        }
        require('./view/post.php');
        break;

(略)
~~~

