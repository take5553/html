# 統計学

参考：[米国データサイエンティストが教える統計学超入門講座【Pythonで実践】 | Udemy](https://www.udemy.com/course/python-stats/)

## 必要ライブラリ

* `seaborn`
* `numpy`
* `scipy.stats`
* `matplotlib.pyplot`

## 準備

1. `seaborn`を用いてチップ支払いに関するデータを読み込み、変数`df`に代入せよ。
2. `df`に`tip_rate`という新カラムを、`tip / total_bill`という計算で作成せよ。

## 記述統計

### 分布の描画

1. `tip`のヒストグラムを`seaborn`を用いて描け。
2. `time`の棒グラフを`seaborn`を用いて描け。

### 平均値

1. `tip_rate`の平均値を`numpy`を用いて求めよ。
2. `tip_rate`の平均値を`df`のメソッドを用いて求めよ。
3. 性別ごとの`tip_rate`の平均値を求めよ。
4. 性別ごとの`tip_rate`の平均値の棒グラフを`seaborn`を用いて描け。
5. 性別ごとの`tip_rate`の平均値の棒グラフを`df`のメソッドを用いて描け。

### 中央値

1. `tip_rate`の中央値を`numpy`を用いて求めよ。
2. `tip_rate`の中央値を`df`のメソッドを用いて求めよ。
3. `tip_rate`が外れ値を持っているかどうか確かめよ。
4. 性別ごとの`tip_rate`の中央値を求めよ。
5. 性別ごとの`tip_rate`の中央値の棒グラフを`seaborn`を用いて描け。
6. `tip_rate`の降順で`df`をソートせよ。

### 最頻値

1. `size`の最頻値を`scipy.stats`を用いて求めよ。
2. `size`の最頻値を`df`のメソッドを用いて求めよ。

### 範囲

1. `tip`の最小値を`numpy`を用いて求めよ。
2. `tip`の最小値を`df`のメソッドを用いて求めよ。
3. 性別ごとの`tip`の最小値を求めよ。
4. `tip`の最大値を`numpy`を用いて求めよ。
5. `tip`の最大値を`df`のメソッドを用いて求めよ。
6. 性別ごとの`tip`の最大値を求めよ。
7. `tip`の範囲を求めよ。

### 四分位数

1. `numpy`を用いて`tip`の四分位数を求めよ。
2. `df`のメソッドを用いて`tip_rate`の四分位数を求めよ。
3. `stats`のメソッドを用いて`tip_rate`の四分位範囲を求めよ。

### 箱ひげ図

1. `matplotlib`を用いて`tip_rate`の箱ひげ図を描け。
2. `seaborn`を用いて、性別ごとの`tip_rate`の箱ひげ図を描け。

### 分散と標準偏差

1. `numpy`を用いて`tip_rate`の分散を求めよ。
2. `numpy`を用いて`tip`の標準偏差を求めよ。

## 2変数間の記述統計

### 共分散

1. `numpy`を用いて`total_bill`と`tip`の共分散行列を求めよ。（普遍共分散にならないように注意）
2. `numpy`を用いて`total_bill`、`tip`、`size`の共分散行列を求めよ。

### 相関係数

1. `numpy`を用いて`total_bill`と`tip`の相関係数行列を求めよ。
2. `numpy`を用いて`total_bill`、`tip`、`size`の相関係数行列を求めよ。
3. `df`のメソッドを用いて相関係数行列を求めよ。
4. `seaborn`を用いて上記3.のヒートマップを表示せよ。

### 連関と連関係数

1. `pandas`を用いて`sex`と`time`の分割表を求めよ。
2. `stats`を用いて`sex`と`time`のカイ2乗値と期待度数を求めよ。
3. クラメールの連関係数を求めるメソッド（`cramers_v(x, y)`）を作れ。
