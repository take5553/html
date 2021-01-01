# オブジェクト指向エクササイズをC#でやってみる

[オブジェクト指向エクササイズをやってみる \- Qiita](https://qiita.com/opengl-8080/items/6f0a458df9c34eccf76c)

↑をなぞる。Javaでされているので、それをあえてC#でやってみる。

[yasuabe blog: オブジェクト指向エクササイズの陳腐化](http://yasutech.blogspot.com/2019/02/scala.html)

批判も入れつつ本当に実践で使える考え方は何かを考える。

基本的には丁寧な解説は残さない。順番と感想を残していく。

## やってみた順

### 名前を省略しない

IDEの「変数の名前変更機能」を使えば簡単。でも省略してもいいもの（抽象度が高くて名前が付けにくいとか）は無理に名づけをしない、あるいは簡単な名前にする。

### 全てのプリミティブ型と文字列型をラップすること

このときに色々やってしまうと先に進まなくなる可能性がある。単純にラップするのがいいのではないか。

また、クラスを作成すると`List<T>`や`Stack<T>`が使えるところが見えてくるかもしれないので、その時は素直にとりあえず作ること。

### ファーストクラスコレクションを使用すること

できた`List<T>`や`Stack<T>`をクラスでラップする。
