# とりあえず使ってみる

## 準備

適当にディレクトリを作成してその中で適当にファイルを作る。ローカルに以下の構成で作成。

~~~
python
└── test
    └── test1.txt

1 directory, 1 file
~~~

`test1.txt`の中身は

~~~
Hello World!
~~~

## Gitのローカルリポジトリ作成

`test`ディレクトリの中に移動してから

~~~shell
$ git init
$ git add .
$ git commit -m "first commit"
~~~

## GitLab上で空のプロジェクトを作成

「Projects」→「Create blank project」

![image-20210508083714740](image/use_gitlab2/image-20210508083714740.png)

作成画面はGitHubとよく似ている。

![image-20210508084038646](image/use_gitlab2/image-20210508084038646.png)

できた。

![image-20210508084247730](image/use_gitlab2/image-20210508084247730.png)

## GitLabにプッシュ

~~~shell
$ git remote add origin http://file-sv-gitlab/takeshi/test1.git
$ git push origin master
~~~

このあとGitLabにログインするときのUsernameとPasswordを聞かれた。

## GitLab上でどうなっているか確認

リロードしてみた。

![image-20210508084957759](image/use_gitlab2/image-20210508084957759.png)

いいじゃん。

## 変更をコミットしてプッシュ

`test1.txt`の内容を以下に変更。

~~~
Hello GitLab!
~~~

コミット＆プッシュ

~~~shell
$ git add .
$ git commit -m "second commit"
$ git push origin master
~~~

またユーザー名とパスワードを聞かれた。これは後で何とかしよう。

GitLabを見てみると、ちゃんと「second commit」に変わっている。

![image-20210508091538546](image/use_gitlab2/image-20210508091538546.png)

Historyを見てみる。「second commit」のところをクリック。

![image-20210508091706806](image/use_gitlab2/image-20210508091706806.png)

ちゃんと変更点がハイライトされている。

![image-20210508091829955](image/use_gitlab2/image-20210508091829955.png)

## 一人開発ならこれで十分

ローカルで開発してコミットしてプッシュを繰り返せば良い。

そもそも一人開発ならホスティングサービスなんていらんけど。まあそこは仕事の都合とか色々。

