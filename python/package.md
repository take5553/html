# パッケージについて

## 概要



[Python基礎講座\(14 モジュールとパッケージ\) \- Qiita](https://qiita.com/Usek/items/86edfa0835292c80fff5)
[【python】pipとは？コマンド一覧と使い方を実例で解説 \- Qiita](https://qiita.com/yuta-38/items/730bf91526f92fe0b41a)

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

（※`apt`を使う意味は後述）

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

## PyPI

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



## `pip`と`pipenv`の違い



[4\.5\. pipenv \- ゼロから学ぶ Python](https://rinatz.github.io/python-book/ch04-05-pipenv/)
[Pipenvを使ったPython開発まとめ \- Qiita](https://qiita.com/y-tsutsu/items/54c10e0b2c6b565c887a)

## `pip`と`apt`の違い



[apt\-getインストールとpipインストール](https://qastack.jp/ubuntu/431780/apt-get-install-vs-pip-install)
[【パッケージ管理】aptとpipのインストール先と内部処理 \- Qiita](https://qiita.com/obukoh/items/f6828d3088a9fe5c71c8)
[UbuntuでPythonのモジュールをインストールしたいとき、aptとpipのどちらを使って入れようか？と思った話](https://yutarine.blogspot.com/2018/07/ubuntu-python-apt-pip.html#:~:text=%E3%81%BE%E3%81%9Fapt%E3%81%A8pip%E3%81%A7%E3%81%AF,pip%E3%81%AF%E6%AC%A0%E3%81%8B%E3%81%9B%E3%81%BE%E3%81%9B%E3%82%93%E3%80%82)

## ライブラリインストール例



`pypdf2`
[Pythonライブラリのインストール － pipの使い方 \| ガンマソフト株式会社](https://gammasoft.jp/python/python-library-install/)

`twython`
[Raspberry Pi2 \+ Twython でニュース bot を作ろう（step2：Twython インストールと、サンプルプログラムの実行）｜NEWS｜株式会社INDETAIL（インディテール）](https://www.indetail.co.jp/blog/raspberry-pi2-twython-newsbot2/)

`python-rpi.gpio`
[プログラミング初心者でも簡単！Raspberry Pi \+ PythonでLチカさせる方法｜NEWS｜株式会社INDETAIL（インディテール）](https://www.indetail.co.jp/blog/8947/)

その他いろいろ
[Pythonで自分のどんなツイートが「いいね」されやすいかを分析してみた \| パソコン工房 NEXMAG](https://www.pc-koubou.jp/magazine/24224)

`pyserial`
[Raspberry Piでシリアル通信試してみた \- Qiita](https://qiita.com/Choirin/items/7bd85786c3c7fda7e1b9)

`pipenv`経由で`pyserial`
[【Python】pySerial を用いたシリアル通信（ループバック試験） \- 7839](https://serip39.hatenablog.com/entry/2020/07/03/070000)
[Raspberry Pi \+ PySerialでシリアル通信 \- Qiita](https://qiita.com/macha1972/items/4869b71c14d25fa5b8f8)

