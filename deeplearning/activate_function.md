 # 活性化関数について

活性化関数はニューラルネットワークの各層（中間層または出力層）にセットする。入力をアフィン変換した後、活性化関数を通して次の層への出力（または最終的な予測）を決める。

## アフィン変換

普通の行列の積と和の計算。

* `W`：重み（中身は計算前にすでに決定されている）
* `b`：バイアス（上に同じ）

~~~python
def Affine(x):
	return np.dot(x, W) + b
~~~

計算グラフによる誤差逆伝播法の実装

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

## 中間層に使われる活性化関数

### ステップ関数

~~~python
def step_function(x):
    y = x > 0
    return y.astype(np.int)
~~~

### シグモイド関数

~~~python
def sigmoid(x):
    return 1 / (1 + np.exp(-x))
~~~

計算グラフによる誤差逆伝播法の実装

~~~python
class Sigmoid:
    def __init__(self):
        self.out = None
    
    def forward(self, x):
        out = 1 / (1 + np.exp(-x))
        self.out = out
        return out
    
    def backward(self, dout):
        dx = dout * (1.0 - self.out) * self.out
        return dx
~~~

### ReLU関数

~~~python
def relu(x):
    return np.maximum(0, x)
~~~

計算グラフによる誤差逆伝播法の実装

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

## 出力層に使われる活性化関数

層としては恒等関数やソフトマックス関数で出力して終わりだけど、学習では層の活性化関数に応じて使う損失関数が決まってくるらしいので、ここではセットで紹介する。

### 恒等関数

入力をそのまま出力に渡す。評価関数に2乗和誤差を使うなら出力層は恒等関数が良い。

回帰問題（回答が連続的な値）に使われる。

#### 損失関数：2乗和誤差

~~~python
def mean_squared_error(y, t):
    return 0.5 * np.sum((y - t)**2)
~~~

### ソフトマックス関数

~~~python
def softmax(x):
    c = np.max(x, axis=-1, keepdims=True)
    exp_a = np.exp(x - c)
    sum_exp_a = np.sum(exp_a, axis=-1, keepdims=True)
    y = exp_a / sum_exp_a
    return y
~~~

入力を正規化（和が1になる形に）して出力。この関数を使うなら評価関数に交差エントロピー誤差を使う。

分類問題（回答が離散的な値）に使われる。

#### 損失関数：交差エントロピー誤差

`y`：予測の結果

* `t`（教師データ）がone-hot表現のとき

  ~~~python
  def cross_entropy_error(y, t):
      if y.ndim == 1:
          t = t.reshape(1, t.size)
          y = y.reshape(1, y.size)
      
      batch_size = y.shape[0]
      return -np.sum(t * np.log(y + 1e-7)) / batch_size
  ~~~

* `t`（教師データ）がラベルのとき

  ~~~python
  def cross_entropy_error(y, t):
      if y.ndim == 1:
          t = t.reshape(1, t.size)
          y = y.reshape(1, y.size)
      
      batch_size = y.shape[0]
      return -np.sum(np.log(y[np.arange(batch_size), t] + 1e-7)) / batch_size
  ~~~

ソフトマックスとその評価関数である交差エントロピー誤差を組み合わせると、計算グラフでの誤差逆伝播法の実装は非常に簡潔になる。

~~~python
class SoftmaxWithLoss:
    def __init__(self):
        self.loss = None
        self.y = None
        self.t = None
        
    def forward(self, x, t):
        self.t = t
        self.y = softmax(x)
        self.loss = cross_entropy_error(self.y, self.t)
        return self.loss
    
    def backward(self, dout=1):
        batch_size = self.t.shape[0]
        dx = (self.y - self.t) * (dout / batch_size)
        return dx
~~~

