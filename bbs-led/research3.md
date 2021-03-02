# 事前調査　～`mysql-connector-python`の使い方～

[Python \+ mysql\-connector\-python の使い方まとめ \- Qiita](https://qiita.com/valzer0/items/2f27ba98397fa7ff0d74)
[MySQL :: MySQL Connector/Python Developer Guide](https://dev.mysql.com/doc/connector-python/en/)
[【mysql\-connector\-python】PythonからMySQLを操作する \- footmark](http://yuk.hatenablog.com/entry/2014/07/07/235408)

もうこれで十分。

## インストール

必要ならば仮想環境に入っておく。仮想環境については[こちら](../python/environment.html)

~~~shell
$ python3 -m pip install mysql-connector-python
~~~

## 使い方

例えば`sample.py`という名前で以下を作成。ポートはデフォルトでは`3306`と思われる。調べ方はこの記事の最後で。

~~~python
import mysql.connector as mydb

# コネクションの作成
connect = mydb.connect(
    host='hostname',
    port='3306',
    user='bbs',
    password=(パスワード),
    database='bbs',
    charset='utf8'
)
cursor = connect.cursor()

cursor.execute('select * from posts')
row = cursor.fetchone()

for i in row:
    print(i)
    
cursor.close()
connect.close()
~~~

## 実行

~~~shell
$ python3 sample.py
~~~

すると以下が表示される。

~~~
1
たけし

テスト
$2y$10$zMVjeAcV8mSo5.gsQIGhg.Rgbu1l.Mmw2B9oGpbmKopR4vj9UVc.6
2021-02-19 22:43:57
2021-02-19 22:43:57
None
~~~

`cursor.fetchone()`を実行したので1レコードだけ取ってきたと思われる。

## MySQLのポートの調べ方

MySQLにログイン。

~~~shell
$ sudo mysql
~~~

以下を打つ。

~~~mysql
> show variables like 'port';
~~~

そうすると以下が出る。

~~~
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| port          | 3306  |
+---------------+-------+
1 row in set (0.006 sec)
~~~

