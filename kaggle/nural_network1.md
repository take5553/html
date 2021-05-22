# ニューラルネットワーク（2層ver）

参考：[ゼロから作るDeep Learningで素人がつまずいたことメモ: まとめ - Qiita](https://qiita.com/segavvy/items/4e8c36cac9c6f3543ffd)

## 概要

実装前。

~~~python
class TwoLayerNet:
    
    params = {}
	layers = OrderedDict()
	#lastLayer
    #learning_rate
    
    def __init__(self, input_size, hidden_size, output_size, learning_rate, weight_init_std=0.01):
        pass
    
    def predict(self, x):
        pass
    
    def loss(self, x, t):
        pass
    
    def gradient(self, x, t):
        pass
    
    def learn(self, x, t):
        pass
    
    def accuracy(self, x, t):
        pass
    
    def numerical_gradient(self, x, t):
        pass
~~~

## メソッド

### コンストラクタ

* `__init__`

  コンストラクタで、

  * 入力層のニューロンの数
  * 隠れ層のニューロンの数
  * 出力層のニューロンの数
  * `learn`実行時の学習率
  * 重み初期化時のパラメーター（省略時は`0.01`）

  を入力。

### 予測と学習に必要なメソッド

* `predict`

  与えられたデータ`x`に対して正解の予測を戻す。

* `loss`

  `predict`を実行し正解の予測を得て、それと教師データ`t`を使って損失関数を実行、結果を戻す。

* `gradient`

  `loss`を実行し損失を得て、それを使って各レイヤーの逆伝播を実行、出力された勾配を戻す。

* `learn`

  `gradient`を実行し勾配を得て、それと学習率を使用して、重みとバイアスを更新する。

学習時は`learn`をループで回す。本番では`predict`で正解を求める。

`loss`はprivate扱いをしてもいい。

### 予測と学習に無関係なメソッド

* `accuracy`

  `predict`を実行して正解の予測を得て、教師データと照らし合わせ、（正解数）/（総数）を出力する。

* `numberical_gradient`

  数値微分で勾配を求め、それを戻す。（`gradient`が正しいかどうか確認する目的で使用）


## フィールド

* `params`

  アフィンレイヤー（行列の内積を計算するレイヤー）の重みとバイアスを格納。

* `layers`

  レイヤーオブジェクト（後述）を格納。

* `lastLayer`

  出力レイヤーオブジェクト（後述）を格納。

  出力層だけ特別にしている理由は、逆伝播法では出力層の活性化関数と損失関数はセットにしといた方がよくて（微分の結果がシンプルになる）、他のレイヤーとごっちゃにしてしまうと予測のときに損失まで計算しちゃうから。

* `leraning_rate`

  学習率。`learning`メソッドでのみ使用。

## レイヤーオブジェクト

層は内部に2つのレイヤーを持つ。

1. アフィンレイヤー
2. 活性化関数レイヤー

入力→アフィンレイヤー→活性化関数レイヤー→出力

レイヤーは`forward`メソッドと`backward`メソッドを持つ。

今回は2層だけで構成するので

* 中間層・・・アフィンレイヤー→ReLUレイヤー
* 出力層・・・アフィンレイヤー→Softmaxレイヤー（→損失関数）

とする。

ただし、先にも述べたようにSoftmaxレイヤーは損失関数とセットにする方が良いので、

* `layers`には第1アフィンレイヤー、ReLUレイヤー、第2アフィンレイヤー
* `lastLayer`にはSoftmaxレイヤー＆損失関数

という組み合わせで格納する。

今後も出力層は概念としてはアフィンレイヤー＋活性化関数レイヤーを持つことになるが、実装としては活性化関数レイヤーと損失関数をセットにする。これを出力層レイヤーと呼ぶ。

各レイヤーの実装は[以前](activate_function.html)に紹介しているけど、改めて紹介。

### アフィンレイヤー

~~~python
class Affine:
    def __init__(self, W, b):
        self.W = W
        self.b = b
        self.x = None
        self.dW = None
        self.db = None
        
    def forward(self, x):
        self.x = x
        out = np.dot(x, self.W) + self.b
        return out
    
    def backward(self, dout):
        dx = np.dot(dout, self.W.T)
        self.dW = np.dot(self.x.T, dout)
        self.db = np.sum(dout, axis=0)
        return dx
~~~

### ReLUレイヤー

~~~python
class Relu:
    def __init__(self):
        self.mask = None
    
    def forward(self, x):
        self.mask = (x <= 0)
        out = x.copy()
        out[self.mask] = 0
        return out
    
    def backward(self, dout):
        dout[self.mask] = 0
        dx = dout
        return dx
~~~

### 出力層レイヤー：SoftmaxWithLossレイヤー

~~~python
class SoftmaxWithLoss:
    def __init__(self):
        self.loss = None
        self.y = None
        self.t = None
        
    def forward(self, x, t):
        self.t = t
        self.y = self.softmax(x)
        self.loss = self.cross_entropy_error(self.y, self.t)
        return self.loss
    
    def backward(self, dout=1):
        batch_size = self.t.shape[0]
        dx = (self.y - self.t) * (dout / batch_size)
        return dx
    
    def softmax(self, x):
        c = np.max(x, axis=-1, keepdims=True)
        exp_a = np.exp(x - c)  # オーバーフロー対策
        sum_exp_a = np.sum(exp_a, axis=-1, keepdims=True)
        y = exp_a / sum_exp_a
        return y
    
    #t:one-hot
    def cross_entropy_error(self, y, t):
        if y.ndim == 1:
            t = t.reshape(1, t.size)
            y = y.reshape(1, y.size)

        batch_size = y.shape[0]
        return -np.sum(t * np.log(y + 1e-7)) / batch_size
~~~

`softmax`と`cross_entropy_error`は外に出しても良い。

## 実装

`TwoLayerNet`を実装する。レイヤーオブジェクトは別ファイル`layers`に書き出して`import`するものとする。また、`numerical_gradient`の計算部分も別ファイル`functions`に書き出して`import`する。（後述）

~~~python
from layers import *
from functions import numerical_gradient
from collections import OrderedDict
import numpy as np

class TwoLayerNet:
    
    params = {}
	layers = OrderedDict()
    
    def __init__(self, input_size, hidden_size, output_size, learning_rate, weight_init_std=0.01):
        self.params['W1'] = weight_init_std * np.random.randn(input_size, hidden_size)
        self.params['b1'] = np.zeros(hidden_size)
        self.params['W2'] = weight_init_std * np.random.randn(hidden_size, output_size)
        self.params['b2'] = np.zeros(output_size)
        
        self.layers['Affine1'] = Affine(self.params['W1'], self.params['b1'])
        self.layers['Relu'] = Relu()
        self.layers['Affine2'] = Affine(self.params['W2'], self.params['b2'])
        
        self.lastLayer = SoftmaxWithLoss()
        self.learning_rate = learning_rate
    
    def predict(self, x):
        for layer in self.layers.values():
            x = layer.forward(x)
        return x
    
    def loss(self, x, t):
        y = self.predict(x)
        return self.lastLayer.forward(y, t)
    
    def gradient(self, x, t):
        self.loss(x, t)
        dout = 1
        dout = self.lastLayer.backward(dout)
        layers = list(self.layers.values())
        layers.reverse()
        for layer in layers:
            dout = layer.backward(dout)
        grads = {}
        grads['W1'] = self.layers['Affine1'].dW
        grads['b1'] = self.layers['Affine1'].db
        grads['W2'] = self.layers['Affine2'].dW
        grads['b2'] = self.layers['Affine2'].db
        return grads
    
    def learn(self, x, t):
        grads = self.gradient(x, t)
        self.params['W1'] -= self.learning_rate * grads['W1']
        self.params['b1'] -= self.learning_rate * grads['b1']
        self.params['W2'] -= self.learning_rate * grads['W2']
        self.params['b2'] -= self.learning_rate * grads['b2']
    
    def accuracy(self, x, t):
        y = self.predict(x)
        y = np.argmax(y, axis=1)
        if t.ndim != 1:
            t = np.argmax(t, axis=1)
        accuracy = np.sum(y == t) / float(x.shape[0])
        return accuracy
    
    def numerical_gradient(self, x, t):
        loss_W = lambda W: self.loss(x, t)
        grads = {}
        grads['W1'] = numerical_gradient(loss_W, self.params['W1'])
        grads['b1'] = numerical_gradient(loss_W, self.params['b1'])
        grads['W2'] = numerical_gradient(loss_W, self.params['W2'])
        grads['b2'] = numerical_gradient(loss_W, self.params['b2'])
        return grads
~~~

`functions.py`

~~~python
import numpy as np

def numerical_gradient(f, x):
    h = 1e-4 # 0.0001
    grad = np.zeros_like(x)
    
    it = np.nditer(x, flags=['multi_index'])
    while not it.finished:
        idx = it.multi_index
        tmp_val = x[idx]
        x[idx] = tmp_val + h
        fxh1 = f(x) # f(x+h)
        
        x[idx] = tmp_val - h 
        fxh2 = f(x) # f(x-h)
        grad[idx] = (fxh1 - fxh2) / (2*h)
        
        x[idx] = tmp_val # 値を元に戻す
        it.iternext()   
        
    return grad
~~~

## 実装が正しいかどうか確認

数値微分で求める勾配と誤差逆伝播法で求める勾配を比較する。

使用するデータはMNISTで、[GitHub - oreilly-japan/deep-learning-from-scratch: 『ゼロから作る Deep Learning』(O'Reilly Japan, 2016)](https://github.com/oreilly-japan/deep-learning-from-scratch)から`mnist.py`を借用する。

~~~python
from mnist import load_mnist
from twolayernet import TwoLayerNet
import numpy as np

(x_train, t_train), (x_test, t_test) = load_mnist(normalize=True, one_hot_label=True)
network = TwoLayerNet(input_size=784, hidden_size=50, output_size=10, learning_rate=0.1)

x_batch = x_train[:3]
t_batch = t_train[:3]

grad_numerical = network.numerical_gradient(x_batch, t_batch)
grad_backprop = network.gradient(x_batch, t_batch)

for key in grad_numerical.keys():
    diff = np.average( np.abs(grad_backprop[key] - grad_numerical[key]))
    print(key + ":" + str(diff))
~~~

出力が以下。

~~~
W1:3.656435397525128e-10
b1:2.063888007807868e-09
W2:6.301500123914619e-09
b2:1.3986855674774644e-07
~~~

なかなかいい誤差じゃないか。
