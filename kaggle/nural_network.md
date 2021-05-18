# ニューラルネットワークの作り方

## 2層だけのネットワーク

### 概要

インターフェイスっぽいものから。

~~~python
class TwoLayerNet:
    
    def __init__(self, input_size, hidden_size, output_size, learning_rate, weight_init_std=0.01):
        pass
    
    def predict(self, x):
        pass
    
    def loss(self, x, t):
        pass
    
    def gradient(self, x, t):
        pass
    
    def update(self, grad, rate):
        pass
    
    def accuracy(self, x, t):
        pass
    
    def numerical_gradient(self, x, t):
        pass
~~~

#### コンストラクタ

* `__init__`

  コンストラクタで、

  * 入力層のニューロンの数
  * 隠れ層のニューロンの数
  * 出力層のニューロンの数
  * `learn`実行時の学習率
  * 重み初期化時のパラメーター（省略時は`0.01`）

  を入力。

#### 予測と学習に必要なメソッド

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

#### 予測と学習に無関係なメソッド

* `accuracy`

  `predict`を実行して正解の予測を得て、教師データと照らし合わせ、（正解数）/（総数）を出力する。

* `numberical_gradient`

  数値微分で勾配を求め、それを戻す。（`gradient`が正しいかどうか確認する目的で使用）

  