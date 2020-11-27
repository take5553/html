# Pythonの環境について

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

## `pip`コマンド

仮想環境が構築できたら遠慮なくパッケージをどんどこ入れていくけど、その時に使うコマンドが`pip`コマンド。ただし、`python`コマンドがPython2を起動してしまうのと同じで、`pip`コマンドはPython2用のパッケージを管理するものっぽい。Python3用のは`pip3`コマンド。

ただし、直接`pip3`コマンドを叩くのではなく、

~~~shell
$ python3 -m pip install (パッケージ名)
~~~

というように、先頭を`python3 -m pip`という形にする方がいいらしい。なぜかは知らない。

> python3 コマンドを使わず、pip3 コマンドを使っても実行できますが、初心者の方には python3 -m pip ... の形式をおすすめします。
>
> [pip \- python\.jp](https://www.python.jp/install/ubuntu/pip.html)

でも、そもそも`pip`コマンドを使うのではなく、`apt`コマンドを使いましょうとも書いてある。

> Ubuntuは、OSが標準でPythonを提供しています。この、OSが提供するPythonの実行環境用に、Ubuntuは独自にカスタマイズした pip コマンドも提供しています。
>
> Ubuntu版の pip は、次のコマンドでインストールできます。
>
> ```
> $ sudo apt install python3-pip
> ```
>
> この方法でインストールした pip コマンドは、OSが提供するPython環境のパッケージを操作します。通常、Ubuntuの実行環境は apt コマンドで操作しますが、pip コマンドでPyPIなどからのパッケージをインストールすると、Ubuntuのパッケージ管理に不具合が発生する可能性があります。
>
> 通常は、pip コマンドではなく、apt コマンドを使って、Ubuntuのパッケージ管理の仕組みを使ってパッケージを更新するようにしましょう。
>
> [pip \- python\.jp](https://www.python.jp/install/ubuntu/pip.html)

「インストールできます」って言ってるけど、挙動から考えるとどうもデフォルトで入っている`pip`がUbuntu（というよりDebian系）版`pip`みたい。

> また、Ubuntu版の pip は、以下の点でオリジナルの pip と異なりますので注意してください。
>
> 通常、pipコマンドでパッケージをインストールすると、/usr/local/lib/ などの、全ユーザが参照できる、Pythonの共有モジュール格納ディレクトリにインストールされます。
>
> しかし、--user オプションを指定すると、~/.local/ などのそれぞれのユーザ個別の専用ディレクトリにインストールされ、他のユーザに影響を与えずに自由にパッケージのインストールや削除ができます。
>
> Debian パッケージの pip コマンド (python3-pip/python-pip) では、--user オブションのデフォルト値が変更されており、特権ユーザではない、一般ユーザとして pip install コマンドを実行すると、自動的に --user オプションが指定されたのもとして実行します。
>
> このため、Ubuntu版の pip コマンドを使用する場合、非特権ユーザが pip install を実行すると、ホームディレクトリの ~/.local ディレクトリにインストールされます。
>
> sudo pip3 install xxx のように、特権ユーザとしてコマンドを実行した場合には、/usr/local/lib にインストールされます。
>
> [pip \- python\.jp](https://www.python.jp/install/ubuntu/pip.html)

ネットの記事に飛びついてうっかり`pip`をアップグレード（しかもpython2用）をしてしまったけど、インストールされたフォルダが`takeshi`ユーザーのホームディレクトリになっている。

~~~shell
$ pip install --upgrade pip
$ pip -V
WARNING: pip is being invoked by an old script wrapper. This will fail in a future version of pip.
Please see https://github.com/pypa/pip/issues/5599 for advice on fixing the underlying issue.
To avoid this problem you can invoke Python with '-m pip' instead of running pip directly.
pip 20.2.4 from /home/takeshi/.local/lib/python2.7/site-packages/pip (python 2.7)
~~~

まだ何もしていないPython3用の`pip`コマンドでは

~~~shell
$ python3 -m pip -V
pip 18.1 from /usr/lib/python3/dist-packages/pip (python 3.7)
~~~

となり、`/usr/lib`の中に入っている。

気持ち悪いからPython2用の`pip`も元に戻したら、やっぱり`/home/takeshi`の中にインストールされたっぽい。

~~~shell
$ python -m pip install pip==18.1
$ python -m pip -V
pip 18.1 from /home/takeshi/.local/lib/python2.7/site-packages/pip (python 2.7)
~~~

極めつけはこれ。普通に`pip -V`するのと、`sudo pip -V`するのでは参照するディレクトリが違う。

~~~shell
$ pip -V
pip 18.1 from /home/takeshi/.local/lib/python2.7/site-packages/pip (python 2.7)
$ sudo pip -V
pip 18.1 from /usr/lib/python2.7/dist-packages/pip (python 2.7)
~~~

とにかく、結局`apt`を使うのがいいのか`pip`でいいのか分からない。要調査。

## PyPl

Pythonの記事を見るとちょいちょい見かけるワード。前提知識っぽい。

> pipはPythonの**パッケージを管理するためのツール**になります。
>
> パッケージには
>
> - **公式が配布しているもの**
> - **サードパーティが配布しているもの**
>
> と大きく分けて2つがあります。
>
> サードパーティのパッケージはPyPIというサイトで配布されています。
> 公式サイトURL=> https://pypi.org/
>
> 公式が配布しているものはたいていPythonをインストールする時点で自動的にインストールされますが、サードパーティが配布しているパッケージは別にインストールをする必要があります。
>
> このサードパーティが配布しているパッケージをインストールするために、pipを使います。pipを使うことでパッケージの管理が楽になります。
>
> [【Python入門】pipとは？使い方をわかりやすく解説！ \| 侍エンジニア塾ブログ（Samurai Blog） \- プログラミング入門者向けサイト](https://www.sejuku.net/blog/50417#pip-2)

## まとめ

結局

* Pythonのバージョンは、問題が起きるまで上げるな
* `venv`で仮想環境を作ってそこで開発を始めろ

