# データセットの基本形

データセット＝大量の画像のセット＋それらの正解ラベル

## 前提

* データセットは基本的に、事前にDLしているものとする。ただし、`download`オプションを`True`にしておくとPyTorchがDLしてくれる。

* データセットの置き場所を`root`と呼ぶ。

* ディレクトリ階層は以下のようにする。（例：MNISTとImageNetをDLしておいた場合）

  ~~~
  root
    ├MNIST
    │  └画像データ色々
    └ImageNet
       └画像データ色々
  ~~~

## 書式

データセットクラスからインスタンス化するような形を取る。

~~~python
dataset = torchvision.datasets.XXXX(
    root: str,
	transform: Callable = None,
    target_transform: Callable = None,
	download: bool = False
)
~~~

* `root`：データセットディレクトリへの相対パス。
* `transform`：前処理を記述した関数のセット。省略したときの既定値は`None`。
* `target_transform`：正解ラベルに対する前処理を記述した関数のセット。省略したときの既定値は`None`。
* `download`：新たにダウンロードをするかどうか。省略したときの既定値は`False`（DLしない）。

ただし`root`以外のオプションは全てのデータセットクラスに実装されているというわけではない。

## 例

~~~
/
└python
  ├data
  │  └(中身は空)
  └test.py
~~~

このようなディレクトリ構成であったときに、`test.py`に以下のように書く。

~~~python
import torchvision
dataset = torchvision.datasets.MNIST('data', download=True)
~~~

そうすると、`data`ディレクトリの中に`MNIST`というディレクトリを勝手に作り、さらにその中にMNIST公式からDLしたデータセットを格納してくれて、さらにさらに`dataset[index]`で画像と正解ラベルのタプルが取り出せる。

例えばこのMNISTの例で言うと、

~~~python
print(dataset[0])
~~~



~~~
(<PIL.Image.Image image mode=L size=28x28 at 0x7FB3D4E908>, 5)
~~~

となる。

pillowイメージでは計算しにくいので、前処理としてPyTorchの配列である`tensor`型に変換しておくと良い。前処理については[次の記事](transform.html)へ。
