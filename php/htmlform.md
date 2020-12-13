# Emmetを使ってHTMLの準備

とりあえず簡単なフォームを作成する。

## 環境

- ローカル
  - Windows 10
  - VSCode 1.51.1
  - XAMPP 7.4.13

## 準備

今までVSCodeの拡張機能の実験のために書いた色々なコードは全部消しておく。

ワークフォルダの構造

~~~
bbs
├.vscodeフォルダ
├bbs.code-workspace
└index.php
~~~

`index.php`の中身

~~~php+HTML
<?php
    
~~~

`Start Developing`としてコミットしておく。

![image-20201213101430597](image/htmlform/image-20201213101430597.png)

## フォルダ・ファイルの新規作成

ワークフォルダに、新たに`view`というフォルダを作りその中に`post.php`を新規作成する。VSCode上でやれば楽ちん。

フォルダの作成。

![image-20201213101529929](image/htmlform/image-20201213101529929.png)

ファイルの作成。

![image-20201213101631091](image/htmlform/image-20201213101631091.png)

`post.php`をクリックすると、ファイルが開かれる。

![image-20201213101747470](image/htmlform/image-20201213101747470.png)

## HTMLコード

VSCodeで`!`と打つと候補が現れる。

![image-20201213100851706](image/htmlform/image-20201213100851706.png)

これでエンターを押すと、必要なタグが一気に作成される。

![image-20201213100940512](image/htmlform/image-20201213100940512.png)

これは[Emmet](https://docs.emmet.io/)というツールがVSCodeに付属しているから。どんなコマンドが使えるのかは[Cheat Sheet](https://docs.emmet.io/cheat-sheet/)を見て雰囲気で学ぶ。多すぎるので必要なものを順番に使っていく。

とりあえずレスポンシブは後で考えることにするので、今は消す。`lang`は`ja`。`title`は「たけしのページの掲示板」

~~~html
<!DOCTYPE html>
<html lang="ja">
<head>
    <meta charset="UTF-8">
    <title>たけしのページの掲示板</title>
</head>
<body>
    
</body>
</html>
~~~

保存をしたら、php cs fixerの機能で強制的に間隔を開けられた。まあ`.php`ファイルだしな。

~~~php+HTML
<!DOCTYPE html>
<html lang="ja">

<head>
    <meta charset="UTF-8">
    <title>たけしのページの掲示板</title>
</head>

<body>

</body>

</html>
~~~

## Emmetを使ってHTMLコーディング

### 作成予定のコード

とりあえずの形はこう。

~~~php+HTML
<body>
    <h1>たけしのページの掲示板</h1>
    <!-- 記事入力エリア -->
    <div class="input_area">
        <form action="./index.php" method="post" id="post_form">
            <p>
                名前：<br>
                <input type="text" name="name" id="name">
            </p>
            <p>
                メールアドレス：<br>
                <input type="email" name="email" id="email">
            </p>
            <p>
                本文：<br>
                <textarea name="post_body" id="post_body" cols="30" rows="10">本文を入力してください。</textarea>
            </p>
            <p>
                <input type="hidden" name="eventId" value="save">
                <input type="submit" value="送信">
            </p>
        </form>
    </div>
    <!-- 記事入力エリア終了 -->

    <!-- 記事表示エリア -->
    <div class="posts">
        <div class="post">
            <div class="auther">
                <p>投稿者　表示場所</p>
            </div>
            <div class="post_timestamp">
                <p>投稿日時　表示場所</p>
            </div>
            <div class="post_body">
                <p>記事本文　表示場所</p>
            </div>
        </div>
        <div class="post">
            <div class="auther">
                <p>投稿者　表示場所</p>
            </div>
            <div class="post_timestamp">
                <p>投稿日時　表示場所</p>
            </div>
            <div class="post_body">
                <p>記事本文　表示場所</p>
            </div>
        </div>
    </div>
    <!-- 記事表示エリア終了 -->
</body>
~~~

### 見出し：`h1`

`h1`と打ってエンター（またはTab）を押せば、`<h1></h1>`を自動で入れてくれる。

![image-20201213104345727](image/htmlform/image-20201213104345727.png)

![image-20201213104402885](image/htmlform/image-20201213104402885.png)

### コメント：`c`

`c`と打ってエンター（またはTab）を押せば、`<!--  -->`を入れてくれる。Emmet Abbreviationと表示されているものがEmmetで自動入力してくれるもの。と言っても表示されている候補全部Emmetなんだけど。

![image-20201213104732004](image/htmlform/image-20201213104732004.png)

![image-20201213104812706](image/htmlform/image-20201213104812706.png)

### divタグ：`div.(クラス名)`

divタグは大体クラス名をセットで書くので、CSS的な書き方で`.(クラス名)`と打てばクラスが付与されたdivタグを入れてくれる。

![image-20201213105200070](image/htmlform/image-20201213105200070.png)

![image-20201213105217594](image/htmlform/image-20201213105217594.png)

### formタグ：`form:(メソッド名)#(ID)`

※メソッドとか送信先とかは後で解説。

POST送信する入力フォームを作るには`form:post`と打ちエンターを押す。

![image-20201213105412588](image/htmlform/image-20201213105412588.png)

すると、`action`属性にカーソルが自動で移動するので、送信先ファイルを指定する。

![image-20201213105553076](image/htmlform/image-20201213105553076.png)

`form:post#post_form`と打てばIdまで自動で付与してくれる。

![image-20201213105912885](image/htmlform/image-20201213105912885.png)

![image-20201213105929709](image/htmlform/image-20201213105929709.png)

### 段落：`p`

`p`→`<p></p>`

![image-20201213110741446](image/htmlform/image-20201213110741446.png)

![image-20201213110754213](image/htmlform/image-20201213110754213.png)

### 改行：`br`

`br`→`<br>`

![image-20201213110844301](image/htmlform/image-20201213110844301.png)

![image-20201213111313788](image/htmlform/image-20201213111313788.png)

### 入力欄（汎用：1行）：`inp`

`inp`→`<input type="text" name="" id="">`

![image-20201213111154599](image/htmlform/image-20201213111154599.png)

![image-20201213111210378](image/htmlform/image-20201213111210378.png)

### 入力欄（Email用）：`input:email`

`input:email`→`<input type="email" name="" id="">`

### 入力欄（汎用：複数行）：`textarea`

`textarea`→`<textarea name="" id="" cols="30" rows="10"></textarea>`

### 隠しパラメーター：`input:h`

`input:h`→`<input type="hidden" name="">`

### 送信ボタン：`input:s`

`input:s`→`<input type="submit" value="">`

## 見た目を確認

![image-20201213114238244](image/htmlform/rs-image-20201213114238244.png)

これはひどい。まあ、デザインは後から。

## コミット＆プッシュ

とりあえずここまでをコミットしてGitHubにプッシュしておく。

![image-20201213123313495](image/htmlform/rs-image-20201213123313495.png)