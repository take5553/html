# 練習

## フィボナッチ数列

引数未満のフィボナッチ数列を標準出力に出力する。

タグ：多重代入、`while`ループ、`docstring`

~~~python
def fib(n):
    """n未満のフィボナッチ数を出力する"""
    a, b = 1, 1
    while a < n:
        print(a, end=' ')
        a, b = b, a+b
    print()
~~~

## 素数判定

引数未満の数に対して素数かどうかを判定し、出力する。

タグ：ループに対する`break`と`else`、`range`関数、`print`関数

~~~python
def prime(num):
    """num未満の数について素数かどうか判定し出力する"""
    for n in range(2, num):
        for x in range(2, n):
            if n % x == 0:
                print(n, 'equals', x, '*', n//x)
                break
        else:
            print(n, 'is a prime number')
~~~

