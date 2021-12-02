# 統計学

参考：[米国データサイエンティストが教える統計学超入門講座【Pythonで実践】 | Udemy](https://www.udemy.com/course/python-stats/)

## 必要ライブラリ

* `seaborn`
* `numpy`
* `scipy.stats`

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
