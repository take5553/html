# CIFAR-10

## データセットの概要

[CIFAR-10 and CIFAR-100 datasets](https://www.cs.toronto.edu/~kriz/cifar.html)

## 基本形

~~~python
dataset = torchvision.datasets.CIFAR10('data')
~~~

`data/CIFAR10`ディレクトリからCIFAR-10データセットを引っ張り出してくれる。

前処理無しの1レコードは`(PILイメージ, 正解ラベル)`で、トレーニングセットを指定すると60000レコード、テストセットを指定すると10000レコードある。

## 追加引数

* `download=False`：`True`にするとDLしてくれる。

* `train=True`：通常はトレーニングセットから作ってくれる。`False`にするとテストセットから作ってくれる。

* `transform=None`：PILイメージをどのように前処理するか。

  `torchvision.transforms.ToTensor()`、`torchvision.transforms.Normalize((0.5,), (0.5,))`は必須と思われる。

* `transform_target=None`：正解ラベルをどのように前処理するか。

https://pytorch.org/vision/stable/datasets.html#cifar

https://rightcode.co.jp/blog/information-technology/pytorch-mnist-learning)
