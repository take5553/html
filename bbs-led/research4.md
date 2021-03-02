# 事前調査　～ファイルの読み書き～

[Pythonでファイルの読み込み、書き込み（作成・追記） \| note\.nkmk\.me](https://note.nkmk.me/python-file-io-open-with/)
[ファイル操作をマスターしよう！Pythonでの読み込み・書き込み方法を徹底解説！ \| TechTeacher Blog](https://www.tech-teacher.jp/blog/python-file/)

## 実験

適当に`dbdata.dat`という名前で以下を作成。

~~~
123
~~~

そして同じディレクトリに`sample3.py`という名前で以下を作成。

~~~python
with open('dbdata.dat') as f:
    s = f.read()
    
t = int(s)

u = 125 # max article id

if u > t:
    with open('dbdata.dat', 'w+') as f:
        f.write(str(u))
~~~

保存終了し、実行。

~~~shell
$ python3 sample3.py
~~~

そして`dbdata.dat`を開くと更新されている。

~~~
125
~~~

