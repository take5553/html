# PyTorch編

ここでは主にPyTorchのチートシート的に書くつもり。

## インストール

https://pytorch.org/

## データセット

* [基本形](datasets.html)
* [前処理](transform.html)

| データセット名           | 概要                                                         |
| ------------------------ | ------------------------------------------------------------ |
| [MNIST](mnist.html)      | 手書きの数字の画像                                           |
| [EMNIST](emnist.html)    | 手書きのアルファベットと数字の画像                           |
| [CIFAR-10](cifar10.html) | 飛行機、自動車、鳥、猫、鹿、犬、蛙、馬、船、トラックの画像<br />32x32のカラー画像で、1クラス6000枚 |
| 他多数                   | 必要なときに見返そう（調べるのが面倒になった）               |

* データセットを自作する（作成中）
  * 画像限定
  * 画像とその他の情報を含む

## データローダー

読み込んだデータセットを学習に使いやすいようにバッチサイズで出力したり、シャッフルしてくれたりするやつ。





https://tzmi.hatenablog.com/entry/2020/02/24/202135

https://tzmi.hatenablog.com/entry/2020/02/09/170019#permute-%E6%AC%A1%E5%85%83%E3%81%AE%E5%85%A5%E3%82%8C%E6%9B%BF%E3%81%88

## モデル

## 損失関数

## オプティマイザー

## 参考

[第1回　難しくない！　PyTorchでニューラルネットワークの基本：PyTorch入門 - ＠IT](https://atmarkit.itmedia.co.jp/ait/articles/2002/06/news025.html)
[pytorch超入門 - Qiita](https://qiita.com/miyamotok0105/items/1fd1d5c3532b174720cd)
[Deep Learning with PyTorch: A 60 Minute Blitz — PyTorch Tutorials 1.9.0+cu102 documentation](https://pytorch.org/tutorials/beginner/deep_learning_60min_blitz.html)

