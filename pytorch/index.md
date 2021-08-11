# PyTorch編

ここでは主にPyTorchのチートシート的に書くつもり。

## インストール

https://pytorch.org/

## データセット

### 基本形

#### 作成

~~~python
data_loader = torch.utils.data.DataLoader(dataset, batch_size=4, shuffle=True)
~~~

#### 使用



* [MNIST](mnist.html)
* 

### ImageNet

https://pytorch.org/vision/stable/datasets.html#imagenet

~~~python
imagenet_data = torchvision.datasets.ImageNet('path/to/imagenet_root')
data_loader = torch.utils.data.DataLoader(imagenet_data, batch_size=4, shuffle=True)
~~~



## モデル

## 損失関数

## オプティマイザー

