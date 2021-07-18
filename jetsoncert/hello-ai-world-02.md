# Classifying Images with ImageNet

Dockerコンテナの中の`/jetson-inference/build/aarch64/bin`の中にある`imagenet`というモジュールについて学ぶ。

予め以下を打っておく。

~~~shell
$ cd /jetson-inference/build/aarch64/bin
~~~

## とりあえず実行

`imagenet`の使い方は以下

~~~shell
$ ./imagenet (インプットソース) (アウトプット先)
~~~

画像を渡すと、それが何であるかをラベリングして画像の左上に表示してアウトプット先に出力する。アウトプット先が画像ファイルだった場合は画像ファイルとして保存される。

![https://raw.githubusercontent.com/dusty-nv/jetson-inference/dev/docs/images/imagenet-orange.jpg](image/hello-ai-world-02/rs-imagenet-orange.jpg)

インプットソースとして他にもいくつかの画像が入ったディレクトリを渡したり、動画を渡したり、ビデオデバイスを渡したりすることができる。この辺のオプションは以下を参照。

[jetson-inference/aux-streaming.md at master · dusty-nv/jetson-inference · GitHub](https://github.com/dusty-nv/jetson-inference/blob/master/docs/aux-streaming.md)

ライブでカメラ画像を使用する場合は

~~~shell
$ ./imagenet /dev/video0
~~~

という感じで、アウトプット先を省略すればウィンドウが表示されてリアルタイムに推測する。

## Pythonのコードの中で使用

### コード全体

~~~python
#!/usr/bin/python3

import jetson.inference
import jetson.utils

import argparse

# parse the command line
parser = argparse.ArgumentParser()
parser.add_argument("filename", type=str, help="filename of the image to process")
parser.add_argument("--network", type=str, default="googlenet", help="model to use, can be:  googlenet, resnet-18, ect.")
args = parser.parse_args()

# load an image (into shared CPU/GPU memory)
img = jetson.utils.loadImage(args.filename)

# load the recognition network
net = jetson.inference.imageNet(args.network)

# classify the image
class_idx, confidence = net.Classify(img)

# find the object description
class_desc = net.GetClassDesc(class_idx)

# print out the result
print("image is recognized as '{:s}' (class #{:d}) with {:f}% confidence".format(class_desc, class_idx, confidence * 100))
~~~

これを書けば良い。ただし、このコードは

* 保存はJetson Nano上
* 実行はDockerコンテナ上

ということにしたいので、一旦Dockerコンテナから抜けて、以下を打つ。

~~~shell
$ cd ~/
$ mkdir my-recognition-python
$ cd my-recognition-python
$ touch my-recognition.py
$ chmod +x my-recognition.py
$ wget https://github.com/dusty-nv/jetson-inference/raw/master/data/images/black_bear.jpg 
$ wget https://github.com/dusty-nv/jetson-inference/raw/master/data/images/brown_bear.jpg
$ wget https://github.com/dusty-nv/jetson-inference/raw/master/data/images/polar_bear.jpg 
~~~

そして再度Dockerコンテナを起動。

~~~shell
$ docker/run.sh --volume ~/my-recognition-python:/my-recognition-python
~~~

### Import

~~~python
#!/usr/bin/python3

import jetson.inference
import jetson.utils

import argparse
~~~

これらのモジュールは、Dockerコンテナにすでに入っているのはもちろん、ソースからビルドしたときは`sudo make install`したときに入っているとのこと。

（この「Hello AI World」というリポジトリはこのモジュールを配布するため？）

### 引数のパース

~~~python
# parse the command line
parser = argparse.ArgumentParser()
parser.add_argument("filename", type=str, help="filename of the image to process")
parser.add_argument("--network", type=str, default="googlenet", help="model to use, can be:  googlenet, resnet-18, ect.")
args = parser.parse_args()
~~~

これにより

* `args.filename`：指定ファイル名（今回は画像）
* `args.network`：指定ネットワークモデル

としてユーザーからの入力を取り出せる。

この`argparse`というライブラリはPython標準のものらしい。詳しくは[こちら](https://docs.python.org/ja/3/howto/argparse.html)。

### 画像のロード

~~~python
# load an image (into shared CPU/GPU memory)
img = jetson.utils.loadImage(args.filename)
~~~

ファイル名から画像をロードする。

ロードした`img`の詳細は[jetson-inference/imagenet-example-python-2.md at master · dusty-nv/jetson-inference · GitHub](https://github.com/dusty-nv/jetson-inference/blob/master/docs/imagenet-example-python-2.md#loading-the-image-from-disk)

### ImageNetのロード

~~~python
# load the recognition network
net = jetson.inference.imageNet(args.network)
~~~

ここでImageNetを、ネットワークモデルを指定しながらロードする。

### ImageNetを使う

~~~python
# classify the image
class_idx, confidence = net.Classify(img)

# find the object description
class_desc = net.GetClassDesc(class_idx)
~~~

`.Classify()`メソッドを使えば画像を分類してくれるらしい。

ただし、出力されるのはインデックスとその確率という素っ気ないものなので、そのクラスの説明文を`.GetClassDesc()`メソッドで取得している。

### 出力

~~~python
# print out the result
print("image is recognized as '{:s}' (class #{:d}) with {:f}% confidence".format(class_desc, class_idx, confidence * 100))
~~~

とりあえずここで出力しているけど、実際は得られた`class_idx`、`confidence`、`class_desc`を使って好きなようにすれば良い。

### `imageNet`の詳細

https://rawgit.com/dusty-nv/jetson-inference/python/docs/html/python/jetson.inference.html#imageNet

## ImageNetの出力をカスタマイズする

現状では事前に学習されたものしか認識できないので、こちらで指定したものを認識させるようにする。

学習にはPyTorch（またはTensorFlow）を使用し、推論にはTensorRTを使用する。よって、ここからは学習させるためにPyTorchがインストールされていることを確認しておく。（[参考](https://github.com/dusty-nv/jetson-inference/blob/master/docs/pytorch-transfer-learning.md)）

### データセットの準備

出力をカスタマイズしようと思ったらデータセットが必要。これは一般的にはラベル付けされた画像を大量に準備するということを意味するけど、この`imageNet`ではラベリングは画像をディレクトリ分けをして、ラベルを列挙したテキストファイルを用意すればいいだけらしい。

例えばクラス分けを以下のとおりとする。

~~~
background
brontosaurus
tree
triceratops
velociraptor
~~~

そうするとフォルダ構造は以下。`labels.txt`の中身は上の5行を記入しておく。

~~~
(任意のディレクトリ名)
    ‣ train/
        • background/
        • brontosaurus/
        • tree/
        • triceratops/
        • velociraptor/
    ‣ val/
        • background/
        • brontosaurus/
        • tree/
        • triceratops/
        • velociraptor/
    ‣ test/
        • background/
        • brontosaurus/
        • tree/
        • triceratops/
        • velociraptor/
    ‣ labels.txt
~~~

そしてこれをパッとやってくれるのが`camera-capture`というツール。以下のように起動する。（[参考](https://github.com/dusty-nv/jetson-inference/blob/master/docs/pytorch-collect.md)）

~~~shell
$ camera-capture /dev/video0
~~~

起動後にデータセットディレクトリへのパスと`labels.txt`を指定すれば勝手にディレクトリを作ってくれて、後はカメラに写った映像を、写真を撮る様に保存をしていけばよい。

### 学習

PyTorchを使って学習する。デフォルトではエポック数は35で、データ量にもよるけど数時間かかる。
`--epochs=N`でエポック数を指定できる。

~~~shell
$ cd /jetson-inference/python/training/classification
$ python3 train.py --model-dir=(学習モデルの保存先ディレクトリのパス) (データセットディレクトリへのパス)
~~~

次に、学習したモデルをTensorRT用に変換する。ここではONNXフォーマットを経由する。

~~~shell
$ python3 onnx_export.py --model-dir=(学習モデルの保存先ディレクトリのパス)
~~~

これで学習モデルの保存先ディレクトリに`resnet18.onnx`というファイルができる。TensorRTにはこれを読み込ませて推論させることになる。

ここではResNet-18が出力されるけど、これは`train.py`の時点でモデルの種類を指定しなかったからデフォルトのResNet-18が採用されて、それをONNXフォーマットに変換しただけ。`train.py --arch (モデルの種類)`という風に指定をすると別の種類のモデルが作成される。ちなみに別のモデルは事前にダウンロードされていないといけない。（[参考](https://github.com/dusty-nv/jetson-inference/blob/master/docs/building-repo-2.md#downloading-models)）

### 推論

~~~shell
$ imagenet.py --model=(学習モデルの保存先ディレクトリのパス)/resnet18.onnx --input_blob=input_0 --output_blob=output_0 --labels=(labels.txtへのパス) (入力ソース)
~~~

単に使うだけのコマンドと比べると

* `--model`
* `--labels`

がポイントらしい。`--input_blob`と`--output_blob`はそれぞれ入力レイヤー・出力レイヤーの名前らしい。重要度は不明。省略するとデフォルトでそれぞれ`'data'`、`'prob'`と設定されるらしい。

### 注意事項

#### 更に追加で学習させるとき

学習モデルの保存先に`checkpoint.pth.tar`というものができているけど、これが各エポック終了時に保存される学習モデルらしい。これを指定して学習を再開させると良い。

~~~shell
$ python3 train.py --model-dir=(学習モデルの保存先ディレクトリのパス) --resume (学習モデルの保存先ディレクトリのパス)/checkpoint.pth.tar --start-epoch (再開させたいエポック)
~~~

再開させたいエポックという引数の意義は謎。ソースを読んだ限り表示に使ってそう。

再学習後、再度`onnx_export.py`を実行してONNXフォーマットに変換するけど、ここで出力されたONNXフォーマットと同じディレクトリに拡張子が`.engine`というファイルがあったらそれは削除しておかないといけない。どうもTensorRTがONNXフォーマットを読み込んだときに生成するファイルらしくて、`.engine`ファイルがあればそっちを使うようになっているらしい。無ければ再生成するという仕組み。

このことは直接どこかに書いてあったわけじゃないけど、Hello AI WorldのGitHubページのIssueでの作者の返答（[参考](https://github.com/dusty-nv/jetson-inference/issues/1030#issuecomment-829346074)）や、TensorRTの仕組み（[参考](https://on-demand.gputechconf.com/gtcdc/2017/presentation/dc7172-shashank-prasanna-deep-learning-deployment-with-nvidia-tensorrt.pdf)）から推測。

#### 精度が出ない

以下が考えられる。

* 学習時のエポック数が足りない

  本コースのCat_Dogデータセット（2500枚の画像×2クラス）とResNet-18を使って100エポック回したときの正確さのグラフ。

  ![image-20210718122959943](image/hello-ai-world-02/image-20210718122959943.png)

  分類対象が2つしか無いので、50％というのはつまり最低スコア。

  * 〜10エポック：最低
  * 〜20エポック：最低に張り付かなくなったけどまだ低い
  * 〜30エポック：60％〜70％になってきた（学習の結果が出だした）
  * 〜40エポック：ほぼ70％後半
  * 〜50エポック：80％をマークするようになってきた
  * 〜100エポック：おそらく限界値84％

  こう見るとエポック数が少ないときはばらつきが多く、たまたま回数にそぐわないような好成績を出すこともあるけど、結局のところほぼ動かなくなるぐらいまで回さないと学習が完了したと自信をもっては言えない。

  Jetson Nano（2GB）だと「1学習＝半日〜1日仕事」という感じか。

* データセットの画像が少ない

  よくある比率としては「`train`：`eval`＝8：2」ぐらいっぽい。で、この`eval`にいろいろなパターンの画像を持たせるのがいいっぽい。結果的にどれぐらいの`train`が必要かというのが計算される。

  上記のCat_Dog分類でも

  * `train`：2500枚×2
  * `eval`：500枚×2
  * `test`：100枚×2

  ということで合計6200枚の画像を使っている。そんなに用意しても認識率は80数％止まり。

  参考：[機械学習モデルのデータ分割／評価方法 | 有意に無意味な話](https://starpentagon.net/analytics/dataset_split_evaluation/)

