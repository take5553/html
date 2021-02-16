# Bootstrap 5で見た目を整える　～全体調整～

仕上げ。

## 環境

- ローカル
  - Windows 10
  - XAMPP 7.4.13
  - MariaDB 10.4.17
- リモート
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4
  - MariaDB 10.3.23

## PCでブラウザの最大化をしたとき

![image-20210216233632195](image/bootstrap4/image-20210216233632195.png)

ちょっと見にくいかな？間延び感がすごい。

無理矢理`max-width`を設定してしまう。

`view/post.php`

~~~php+HTML
<body>
    <!-- 以下を追加 -->
    <style>
    .maxWidth{
        max-width: 960px;
    }
    </style>
    <!-- ここまで -->
    <div class="container maxWidth"> <!-- maxWidthクラスを追加 -->
        <div class="row mb-4">
            <h1>たけしのページの掲示板</h1>
        </div>
        <!-- 記事入力エリア -->
~~~

![image-20210216234515649](image/bootstrap4/image-20210216234515649.png)

ちょっとシュッとした。

## タイトルを調整

タイトルだけ凝ってみようかと思ったけど、今更なのでBootstrapのみで。と言っても余白を開けただけ。

![image-20210216235851965](image/bootstrap4/image-20210216235851965.png)

`view/post.php`

~~~php+HTML
<body>
    <style>
    .maxWidth{
        max-width: 960px;
    }
    </style>
    <div class="container maxWidth">
        <div class="row my-5"> <!-- ←mb-4からmy-5に変更 -->
            <h1>たけしのページの掲示板</h1>
        </div>
        <!-- 記事入力エリア -->
~~~

記事編集画面の方は「記事編集」の文字が`<h2>`タグになっていてちょい微妙だったので変更。

`view/edit.php`

~~~php+HTMl
<body>
    <div class="container">

        <div class="row my-5"> <!-- ←post.phpに合わせる -->
            <h1>たけしのページの掲示板</h1>
        </div>
        
        <!-- 以下を追加 -->
        <div class="row mb-3">
            <div class="col">
                <div class="bg-light py-2 h4 text-center border rounded">
                    記事編集
                </div>
            </div>
        </div>
        <!-- ここまで -->
        
        (略)
~~~

![image-20210217003044528](image/bootstrap4/rs-image-20210217003044528.png)

なんとなく記事編集モード感。