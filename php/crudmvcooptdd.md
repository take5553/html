# CRUDとMVCとOOPとTDDについて

これから具体的に掲示板を作っていくけど、その前にベースにする概念について説明する。ただし、正確な解説なんて調べればいっぱい出てくるので、ここでは自分なりの解釈で雑に説明する。

## CRUD

[クラッド \(CRUD\)とは｜「分かりそう」で「分からない」でも「分かった」気になれるIT用語辞典](https://wa3.i-3-i.info/word123.html)

* C・・・Create「作成」
* R・・・Read「読み出し」
* U・・・Update「更新」
* D・・・Delete「削除」

記事を保存するデータベースに対して、これら4つの機能を実装する。

と言っても

* Read・・・掲示板ページにアクセスしたら誰かの投稿が見えた
* Create・・・自分も記事が投稿できる
* Update・・・自分の記事を変更できる
* Delete・・・自分の記事を削除できる

ということであり、考えたら至って普通の機能のこと。

でも、今書きながら気づいたけど、これってUとDを実現しようと思ったら「他人の記事は変更・削除できない」ようにしないといけなくて、それにはユーザーログイン機能か、せめて記事にパスワードを持たせないといけない。で、DB作成時にはそんなこと考えていなかったのでそんなカラムは作っていない・・・まずい・・・

・・・と言うように、たとえ簡単なアプリであっても、ちょっと考えるだけで作るべき具体的な機能が見えてきたりする。

## MVC

[MVCモデルについて \- Qiita](https://qiita.com/s_emoto/items/975cc38a3e0de462966a)

* M・・・Model
* V・・・View
* C・・・Controller

アプリ全体の構造を指す。掲示板というアプリの中身は「M」「V」「C」に分けて作ることにする。

### M

Q：「記事の投稿って、どういう処理で実現する？」

に答えるのがM。

つまり

1. 入力として投稿者名、メアド、本文を受け取って
2. それを元にSQL文を作って
3. そのSQL文をMySQLに投げる

ということ。

ここでMはVではないので、「投稿した記事を追加した形でユーザーに見せる」というところまでは考えてはいけない。

### V

Q：「DBに保存されてる記事をどう表示する？」

に答えるのがV。

ここでは

1. HTMLとかCSSなんかはあらかじめ作っておいて
2. 投稿者名、メアド、本文をDBから受け取ってHTMLに配置して
3. 記事の数だけ2を繰り返す

ということ。

ここでもVはMではないので、「どうやってDBからデータを取ってくるか」ということは考えない。Vはあらかじめ決めた「データの受け取り方」に従って「こうやってコードを書いて置けば誰かがここに適切な文字列を入れといてくれるはずだ」ということだけを考えてコーディングしていく。

WordPressのカスタムテーマがちょうどいい例。`the_title()`と書けばその場所にWordPressが勝手に記事タイトルを文字列として配置してくれ、`the_content()`と書けばその場所に`<p>`タグで囲まれた文字列をWordPressが勝手に配置してくれる。どうやってDBの中からデータを引っ張ってくるのか、ということは全く考えずに見た目を作ることができる。

### C

Cはちょっと説明しにくいけど、あえて言うなら`index.php`。

えっ？

となるかもしれないけど、掲示板やブログは実は毎回すべて`index.php`にアクセスしている。ウチのWordPressは

* `https://arcticstreet.ddns.net/wordpressblog/tonteki/`→トンテキの記事
* `https://arcticstreet.ddns.net/wordpressblog/buri-daikon/`→ぶり大根の記事

が表示されるけど、実はどちらも`https://arcticstreet.ddns.net/wordpressblog/index.php`に

* `tonteki`というパラメーター（とその他もろもろのパラメーター）を渡している

* `buri-daikon`というパラメーター（とその他もろもろのパラメーター）を渡している

と言える。何もパラメーターを渡さなければホーム画面が表示されるというわけ。

つまりどんな場合でも一旦`index.php`にアクセスし、一緒に付いてきたパラメーターを元に「ユーザーは何がしたいのか」を判断し、適切なMを呼び出し処理をさせ、データの準備が整ったら適切なVを呼び出し表示を作らせる、ということをするのがCであり、大体の場合`index.php`がそれにあたる。

規模の大きいアプリだと`index.php`からさらに他のPHPファイルを`require`するかもしれないけど、`index.php`が起点であることはほぼ間違いない。

まあ詳しくは追い追い。

## OOP

[OOPとは \- Google 検索](https://www.google.com/search?q=OOP%E3%81%A8%E3%81%AF&oq=OOP%E3%81%A8%E3%81%AF&aqs=chrome..69i57j0j0i5i30l3.2413j0j7&sourceid=chrome&ie=UTF-8)

* O・・・Object「オブジェクト」
* O・・・Oriented「指向」
* P・・・Programming「プログラミング」

要はコーディングのスタイルの一種。コード整理術。

これに関しては自分も不慣れなところがあるので、作りながら確かめていきたいけど、現時点での認識としては

* 「データ」とそのデータを操作する「処理」をひとまとめにすること
* そのひとまとまりは、あれやこれやを同時に担当せず、単一の動作に対して責任を負う
* そのひとまとまりは、簡単に流用できる
* でも変更をしたときは、影響はなるべくそのまとまりの中だけにとどめる

とかそんな感じ。

一つ言えるのは「ちゃんとしたOOPをするのは難しい」ということ。

## TDD

[TDDとは \- Google 検索](https://www.google.com/search?q=TDD%E3%81%A8%E3%81%AF&oq=TDD%E3%81%A8%E3%81%AF&aqs=chrome..69i57j0l3j0i5i30l4.3555j0j9&sourceid=chrome&ie=UTF-8)

* T・・・Test「テスト」
* D・・・Driven「駆動」
* D・・・Development「開発」

駆動という言葉はちょっと微妙だけど、「テストに引っ張られて進めていく開発」ということであり、具体的に言うと「先にテストを書いて、それが通るように処理を実装していくこと」。余裕があればテストを通した後、その状態を維持しつつさらに改良もする。


