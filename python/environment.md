# Pythonのインストールと仮想環境について

## 環境

- Raspberry Pi
  - Raspberry Pi 3B+
  - Raspberry Pi OS 10.4

## Pythonは最初からインストールされている

~~~shell
$ python -V
Python 2.7.16
$ python3 -V
Python 3.7.3
~~~

Pythonは2系と3系があり、互換性がない。最新版は当然3系。だけど、この3系もマイナーバージョンは最新ではない。（2020/11現在で3.9が最新）

どうもOSもPythonを使っているらしく、Pythonのバージョンをむやみに上げると動作が不安定になるかも。知らんけど。

> pyenvはRubyの文化から来ていますが、そもそもPythonでは処理系のバージョンをあげろ圧力はあまりないです。周りのツールが揃うまではバージョンを上げないで待つ、というのも行われてきたことです。ライブラリも、少し前のバージョンまでサポートするのはよく使うのは当たり前。ライブラリがバージョンアップしたら、Pythonのサポートバージョンが古い方に伸びた、というのも見かけたことがあります。OSがシステムコンポーネントとして使っているのがPython。RedHatの古いバージョンだと未だに2.4バンドルですよね。さすがにライブラリで2.4を積極的にメンテはない(2.6以上じゃないと3系と同時メンテはツライ)ので、そこまで古いバージョンでは古いライブラリ使ってね、とはなりますが。
>
> [pyenvが必要かどうかフローチャート \- Qiita](https://qiita.com/shibukawa/items/0daab479a2fd2cb8a0e7#pyenv%E3%81%8C%E4%B8%80%E8%88%AC%E3%83%A6%E3%83%BC%E3%82%B6%E3%81%AB%E3%81%82%E3%81%BE%E3%82%8A%E5%BF%85%E8%A6%81%E3%81%8C%E3%81%AA%E3%81%84%E7%90%86%E7%94%B1)

> あまりOSを使わないので詳細は不明だが、LinuxではシステムとしてPython(2.x)が入っており、システムから利用されることもある。Anacondaを不用意にインストール・設定することでシステムのPythonより優先的に使用されるようになり、特にPython3を入れた場合など、トラブルの原因となることもあるようだ。
>
> [Pythonインストール\(Anaconda\) \[いかたこのたこつぼ]](https://ikatakos.com/pot/programming/python/install)

Pythonに関しては問題が起きない限りバージョンはそっとしておいた方がいいかもしれない。

## 仮想環境

Pythonは必要に応じてパッケージをどんどん入れたりする。その際、やっぱりバージョンの問題でよく分からないエラーが起こるかもしれないっていうことで、仮想環境を作って目的ごとに専用の環境を作り、そこにパッケージなどを入れていくようにするのが一般的らしい。

> Python を使って開発や実験を行うときは、用途に応じて専用の実行環境を作成し、切り替えて使用するのが一般的です。こういった、一時的に作成する実行環境を、**「仮想環境」** と言います。
>
> [仮想環境 \- python\.jp](https://www.python.jp/install/ubuntu/virtualenv.html)

先述の`pyenv`に関する記事によると`venv`コマンドで仮想環境を作るのが主流らしい。でもネットを探すと`pyenv`で環境構築しているのも見かける。ここでもおとなしく主流の`venv`で仮想環境を構築するのが良い。

## 仮想環境の実験

[仮想環境: Python環境構築ガイド \- python\.jp](https://www.python.jp/install/ubuntu/virtualenv.html)に沿ってやってみる

適当にフォルダを作る。

~~~shell
$ mkdir sample1
$ cd sample1
~~~

仮想環境を作って実行。頭に`(.venv)`と出る。

~~~shell
$ python3 -m venv .venv #少し時間がかかる
$ . .venv/bin/activate
(.venv) $
~~~

これで仮想環境に入ったので、`pyserial`というモジュールをインストールしてみる。

~~~shell
(.venv) $ python3 -m pip install pyserial #少し時間がかかる
~~~

インストールしたパッケージリストを見てみる。

~~~shell
(.venv) $ python3 -m pip list
Package       Version
------------- -------
pip           18.1
pkg-resources 0.0.0
pyserial      3.5
setuptools    40.8.0
~~~

仮想環境から抜ける。

~~~shell
(.venv) $ deactivate
$
~~~

この状態でパッケージリストを見る。

~~~shell
$ python3 -m pip list
Package             Version
------------------- -----------
acme                0.31.0
asn1crypto          0.24.0
astroid             2.1.0
asttokens           1.1.13
...
~~~

全然違うリストが出てきた。Raspberry PiのOSがインストールしたものも含まれてるっぽい。

## まとめ

結局

* Pythonのバージョンは、問題が起きるまで上げるな
* `venv`で仮想環境を作ってそこで開発を始めろ

