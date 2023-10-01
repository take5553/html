# 競技プログラミング

自分用逆引き備忘録

## スニペット

`rmqmax` : Segment Tree (Max)

`rmqmin` : Segment Tree (Min)

`rsq` : Segment Tree (Sum)

`mg` : 重み無しグラフ （DFS＝到達可能性探索、BFS＝最小移動数、メモ化再帰＝最大移動数）

`mwg` : 重み有りグラフ（ダイクストラ＝移動の最小コスト、`shortestPath` ＝最小コストの道順）

`uf` : Union Find（同じグループかどうか）

`mf` : Maximum Flow（最大流量、最小カット）

`bins` : 二分探索

`doubleprintf` : 小数を標準出力に出す時

`bms` : 要素の重複を許した集合を、上位と下位に分ける

`mb` : Board



## Bit全探索

* [各桁が狭義の単調減少になっている数字](abc321c.html)

    狭義の単調減少＝数字は1度しか現れない、かつ順番が一意に決まる

## 二分探索

* [全てのi, jについてmin(a.at(i) + b.at(j) , p)の和](abc321d.html)

    bをソート→iごとに p - a.at(i)をbで二分探索→手前はa + bj、以後はp 

## Board系（ポリオミノ）

* [二枚のポリオミノ a, bを重ね合わせて、理想形xを作る](abc307c.html)

    `MyBoard` クラスを使って、ズラしを全探索

* [3つのポリオミノの敷き詰め可能性](abc322d.html)

    `MyBoard` クラスを使って、回転＆ズラしを全探索

## multiset系

* [n個の配列の内、上位k個の和を求める（入れ替えあり）](abc306e.html)

    2つのmultisetで値を行ったり来たりさせる。