# MNIST

## データセットの概要

[MNISTデータベース - Wikipedia](https://ja.wikipedia.org/wiki/MNIST%E3%83%87%E3%83%BC%E3%82%BF%E3%83%99%E3%83%BC%E3%82%B9)

## 基本形

~~~python
dataset = torchvision.datasets.MNIST('data')
~~~

`data/MNIST`ディレクトリからMNISTデータセットを引っ張り出してくれる。

前処理無しの1レコードは`(PILイメージ, 正解ラベル)`で、トレーニングセットを指定すると60000レコード、テストセットを指定すると10000レコードある。

## 追加引数

* `download=False`：`True`にするとDLしてくれる。

* `train=True`：通常はトレーニングセットから作ってくれる。`False`にするとテストセットから作ってくれる。

* `transform=None`：PILイメージをどのように前処理するか。

  `torchvision.transforms.ToTensor()`、`torchvision.transforms.Normalize((0.5,), (0.5,))`は必須と思われる。

* `transform_target=None`：正解ラベルをどのように前処理するか。

https://pytorch.org/vision/stable/datasets.html#mnist

## 参考

[pyTorchのtransforms,Datasets,Dataloaderの説明と自作Datasetの作成と使用 - Qiita](https://qiita.com/mathlive/items/2a512831878b8018db02#5-1-2-datasets%E3%81%AB%E3%82%88%E3%82%8B%E5%89%8D%E5%87%A6%E7%90%86%E3%83%80%E3%82%A6%E3%83%B3%E3%83%AD%E3%83%BC%E3%83%89)
[pytorch超入門 - Qiita](https://qiita.com/miyamotok0105/items/1fd1d5c3532b174720cd#mnist%E3%81%AE%E3%82%B5%E3%83%B3%E3%83%97%E3%83%AB)
[【PyTorch入門】PyTorchで手書き数字(MNIST)を学習させる – 株式会社ライトコード](https://rightcode.co.jp/blog/information-technology/pytorch-mnist-learning)
