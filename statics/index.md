# 統計学

参考：[米国データサイエンティストが教える統計学超入門講座【Pythonで実践】 | Udemy](https://www.udemy.com/course/python-stats/)

## 必要ライブラリ

* `seaborn`
* `numpy`
* `scipy.stats`
* `matplotlib.pyplot`
* `pandas`
* `sklearn.preprocessing.StandardScaler`
* `statsmodels.stats.proportion.proportions_ztest`
* `statsmodels.api.qqplot`
* `statsmodels.stats.power.TTestIndPower`

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

## 確率

### 確率分布

1. `stats`：1から6までの離散型確率変量を生成せよ。
2. `stats`：0から1までの連続型確率変量を生成せよ。
3. 1から6までの一様分布な確率質量関数のグラフを描け。
4. 0から1までの一様分布な確率密度関数のグラフを、描画範囲を-2から2として描け。

### 累積分布関数

1. -3から3までの正規分布に従う確率密度関数のグラフを描け。
2. -3から3までの正規分布に従う累積分布関数のグラフを描け。
3. 上記6.のSurvival functionのグラフを描け。

### 正規分布

1. -5から15の間で以下の正規分布のグラフを描け。また、標準正規分布も同じグラフに描け。

   | 平均 | 標準偏差 |
   | ---- | -------- |
   | 10   | 3        |
   | 8    | 3        |
   | 8    | 1        |

### カーネル密度推定

1. `tip`のカーネル密度推定のグラフを`stats`を用いて-3から10の範囲で描け。
2. `tip`のカーネル密度推定のグラフを`seaborn`を用いてヒストグラムと同時に描け。

### 68-95-99.7ルール

1. 68-95-99.7ルールを確かめよ。

### 標準化

1. `tip_rate`を標準化したデータを取得せよ。

### 二項分布

1. `n=3, p=1/6`の二項分布の確率質量関数のグラフを描け。
2. `n`を大きくした時に、平均`np`、分散`npq`の正規分布に近似されることをグラフを描いて確かめよ。

## 推測統計

### 標本分布

1. `df`のうち50個の標本を取り`tip`の標本平均を計算することを100回繰り返すとき、標本平均100個の平均と分散をそれぞれ求め、それらが母平均及び母分散/nに近似することを確かめよ。

### 不偏分散

1. `stats`を用いて`tip`の不偏分散を求めよ。
2. `stats`を用いて`tip`の不偏分散の平方根を求めよ。
2. `df`のうち50個の標本をとり、`tip`の標本不偏分散を計算することを100回繰り返し、それら100個の標本不偏分散の平均を求めて、それが母分散に近似することを確かめよ。

### 比率の区間推定

1. 1000人の標本集団において、内閣支持率が60％であった場合の、支持率の区間推定値を求めよ。
2. `df`のうち50個の標本をとり、男性の比率の区間推定値を計算することを100回繰り返したとき、それらの区間に実際に母比率が入っている場合の数を求めよ。

### 平均の区間推定

1. `df`のうち50個の標本をとり、`tip`の平均の区間推定値を正規分布を用いて計算することを100回繰り返したとき、それらの区間に実際の母平均が入っている場合の数を求めよ。
2. 自由度が1, 6, 11であるt分布を描き、同じ図の中に標準正規分布も描け。
2. `df`のうち50個の標本をとり、`tip`の平均の区間推定値を正規分布とt分布を用いて算出し比べよ。

### 比率の差の検定（Z検定とカイ二乗検定）

1. あるWebサイトで訪問者1000人に対してあるバナー広告が30回クリックされたとする（クリック率3.0%）。施策を変えたところ1000人中33回クリックされた（クリック率3.3%）。このことに対しZ検定を行い、クリック率の上昇に有意差があるかどうか確かめよ。ただし有意水準は0.05とし、片側検定を行うものとする。
2. 上記のことをカイ二乗検定を行うことによっても同様の結果が得られることを確かめよ。
3. `df`のうち50個の標本を2つ取り、何か施策を行う前と後と見立てて、`time`の項目の`Dinner`の数に差があるかどうかをZ検定とカイ二乗検定で確かめよ。

### 平均値差の検定

1. `df`を標本と見立てて、男女の`tip_rate`の平均値の差に有意差があるかどうか、ウェルチのt検定で確認せよ。ただし、有意水準は5%とする。

### QQプロット

1. `df`の男女の`tip_rate`のQQプロットをそれぞれ描け。

### シャピロウィルク検定

1. `df`の男女の`tip_rate`の正規性の検定を行って、それぞれの分布が正規分布かどうか確かめよ。ただし外れ値を除外して検定を行うこと。

### F分布

1. `dfn`が1, 6, 11のそれぞれのときに`dfd`が1, 6, 11であるF分布（合計9個）を描け。

### F検定

1. `df`の男女の`tip_rate`が等分散かどうかF検定を行って確かめよ。

### 対応のある平均値差の検定

1. `data/blood_pressure.csv`をpandasのDataFrameとして読み込み、`bp_df`に格納せよ。
2. `bp_df`の`bp_before`と`bp_after`に対して対応のある平均値差の検定を行い、`bp_before`より`bp_after`の方が平均値が低いということを確かめよ。

### Cohen's d

1. Cohen's dを計算する関数を定義せよ。引数はデータ配列を2つ受け取るものとする。
2. 1.で定義した関数を使って、`df`における男女の`tip_rate`の平均について、効果量を測定せよ。

### 検定力分析

1. 上記の効果量を用いて、`df`における男女の`tip_rate`の検定の検定力を求めよ。有意水準は0.05とする。
2. 検定力0.8とするのに必要なサンプルサイズを求めよ。ただし効果量や男女のサンプルサイズの比は1.と同じとする。

### 検定力の推移

1. 効果量が0.2, 0.5, 0.8のときのそれぞれの検定力の推移をグラフに描け。ただし、Number of Observationsは5〜100までとする。

