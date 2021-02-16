# Bootstrapで見た目を整える　～エラーメッセージ部分～

おそらくレア現象となるであろう、セッション切れのエラーメッセージをBootstrapでそれっぽく整える。

## 環境

- ローカル
  - Windows 10
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモート
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - MariaDB 10.3.23

## エラーメッセージ再現方法

セッション切れを手動で起こす。

1. ブレークポイントを設定してデバッグモードにする。

   ![image-20210216231458143](image/bootstrap3/image-20210216231458143.png)

   ![image-20210216231542753](image/bootstrap3/image-20210216231542753.png)
   
2. 適当に記事を投稿して、ブレークポイントで止める。

   ![image-20210216231833577](image/bootstrap3/image-20210216231833577.png)

   ![image-20210216231903194](image/bootstrap3/image-20210216231903194.png)

3. VSCodeの下部にある（はず）デバッグコンソールに`$token = ""`を打つ。

   ![image-20210216232038144](image/bootstrap3/image-20210216232038144.png)

   エンターを押すと実行され`$token`に空文字が代入される。

   ![image-20210216232119497](image/bootstrap3/image-20210216232119497.png)

4. そのままコードを再実行。

   ![image-20210216232208855](image/bootstrap3/image-20210216232208855.png)

5. エラーメッセージが出ているのを確認。

   ![image-20210216232304383](image/bootstrap3/image-20210216232304383.png)

これを整える。

## エラーメッセージ用の装飾

`view/post.php`

~~~php+HTML
<!-- エラーメッセージ表示エリア -->
<?php if (isset($errmsg)) :?>
<div class="errormsg alert alert-danger" role="alert"> <!-- ←変更 -->
    <?php echo h($errmsg) ?> <!-- pタグを取った -->
</div>
<?php endif; ?>
<!-- エラーメッセージ終了 -->
~~~

![image-20210216232653033](image/bootstrap3/rs-image-20210216232653033.png)

おおー。これでいいやん。

ちょっとマージンを設定。`mb-5`をクラスに追加。

![image-20210216232925247](image/bootstrap3/rs-image-20210216232925247.png)

よし。