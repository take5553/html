# EMNIST

## データセットの概要

https://atmarkit.itmedia.co.jp/ait/articles/2009/28/news024.html

## 基本形

~~~python
dataset = torchvision.datasets.EMNIST(root='data', split='mnist')
~~~

`data/EMNIST`ディレクトリからEMNISTデータセットを引っ張り出してくれる。

`split`オプションは必須で、EMNISTの中のどのデータセットを使うかを指定する。

> - EMNIST ByClass: 814,255 characters. 62 unbalanced classes.
> - EMNIST ByMerge: 814,255 characters. 47 unbalanced classes.
> - EMNIST Balanced:  131,600 characters. 47 balanced classes.
> - EMNIST Letters: 145,600 characters. 26 balanced classes.
> - EMNIST Digits: 280,000 characters. 10 balanced classes.
> - EMNIST MNIST: 70,000 characters. 10 balanced classes.
>
> The full complement of the NIST Special Database 19 is available in the ByClass and ByMerge splits. The EMNIST Balanced dataset contains a set of characters with an equal number of samples per class. The EMNIST Letters dataset merges a balanced set of the uppercase and lowercase letters into a single 26-class task. The EMNIST Digits and EMNIST MNIST dataset provide balanced handwritten digit datasets directly compatible with the original MNIST dataset.
>
> [The EMNIST Dataset | NIST](https://www.nist.gov/itl/products-and-services/emnist-dataset)

* `byclass`：フルセット
* `bymerge`：フルセット（クラス分けが違う）
* `balanced`：`bymerge`のクラス間でのサンプル数を同じにしたもの。
* `letters`：アルファベットだけに絞ったもの。（大文字小文字が混在）
* `digits`：数字
* `mnist`：数字

前処理無しの1レコードは`(PILイメージ, 正解ラベル)`。

## 追加引数

* `download=False`：`True`にするとDLしてくれる。

* `train=True`：通常はトレーニングセットから作ってくれる。`False`にするとテストセットから作ってくれる。

* `transform=None`：PILイメージをどのように前処理するか。

  `torchvision.transforms.ToTensor()`、`torchvision.transforms.Normalize((0.5,), (0.5,))`は必須と思われる。

* `transform_target=None`：正解ラベルをどのように前処理するか。

https://pytorch.org/vision/stable/datasets.html#mnist

## 参考

[The EMNIST Dataset | NIST](https://www.nist.gov/itl/products-and-services/emnist-dataset)
[EMNIST：手書きアルファベット＆数字の画像データセット：AI・機械学習のデータセット辞典 - ＠IT](https://atmarkit.itmedia.co.jp/ait/articles/2009/28/news024.html)
