# 調査2　Tkinter

Python標準のGUIライブラリの使い方。

## 基本形

特に決まりは無いけど、「1ファイル1画面」という作り方が分かりやすいのではないか。

~~~python
import tkinter as tk
from tkinter import ttk

class SampleWindow(ttk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(column=0, row=0, sticky=(tk.N, tk.W, tk.E, tk.S))
        self.create_widgets()
        # 初期化処理をここに書く
    
    def create_widgets(self):
        # 画面に配置する子ウィジェットをここで定義する。
        self.frame1 = ttk.Frame(self)
        self.frame1.grid(column=0, row=0, sticky=(tk.W, tk.E), padx=10, pady=10)
        self.button1 = ttk.Button(self.frame1, text="Button1", command=self.command1)
        self.button1.grid()
        
    def command1(self):
        # ボタンを押したときの処理など（クラス外に定義しても可）
        pass
    
if __name__ == "__main__":
    window = tk.Tk()
    app = SampleWindow(master=window)
    app.mainloop()
~~~

## ウィンドウ関係

### サブウィンドウ

核は`tk.Toplevel()`。これでサブウィンドウが作れる。

準備として、サブウィンドウ担当のファイルを作成。

`sub_window1.py`

~~~python
import tkinter as tk
from tkinter import ttk

class SubWindow1(ttk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(column=0, row=0, sticky=(tk.N, tk.W, tk.E, tk.S))
        self.create_widgets()
    
    def create_widgets(self):
        self.frame1 = ttk.Frame(self)
        self.frame1.grid(column=0, row=0, sticky=(tk.W, tk.E), padx=10, pady=10)
        self.button1 = ttk.Button(self.frame1, text="Button1", command=self.command1)
        self.button1.grid()
        
    def command1(self):
        pass
    
if __name__ == "__main__":
    window = tk.Tk()
    app = SubWindow1(master=window)
    app.mainloop()
~~~

そしてメインウィンドウを作成。

`main.py`

~~~python
import tkinter as tk
from tkinter import ttk
from sub_window1 import SubWindow1

class SampleWindow(ttk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(column=0, row=0, sticky=(tk.N, tk.W, tk.E, tk.S))
        self.create_widgets()
        # メンバー変数をここで定義しておかないとボタンを初めて押すときにエラーが出る
        self.sub_window = None
    
    def create_widgets(self):
        self.frame1 = ttk.Frame(self)
        self.frame1.grid(column=0, row=0, sticky=(tk.W, tk.E), padx=10, pady=10)
        self.button1 = ttk.Button(self.frame1, text="Button1", command=self.command1)
        self.button1.grid()
        
    def command1(self):
        # このifが無いとボタンを押すたびにウィンドウが作成されてしまう
        if self.sub_window == None or not self.sub_window.winfo_exists():
            self.sub_window1 = tk.Toplevel()
            self.sub_window_form = SubWindow1(self.sub_window1)
    
if __name__ == "__main__":
    window = tk.Tk()
    app = SampleWindow(master=window)
    app.mainloop()
~~~

こうすることで`SubWindow1`をサブウィンドウとして呼び出せる。また、`python3 sub_window1.py`として起動すればサブウィンドウ単体でも起動できる。

### ウィンドウが閉じられるときの処理

ウィンドウが閉じられる前に色々と処理したいことがあるとき、以下のように初期化時に処理したい内容を登録しておく。ポイントは`master.protocol()`。

~~~python
import tkinter as tk
from tkinter import ttk

class SampleWindow(ttk.Frame):
    def __init__(self, master=None):
        super().__init__(master)
        self.grid(column=0, row=0, sticky=(tk.N, tk.W, tk.E, tk.S))
        self.create_widgets()
        self.master.protocol("WM_DELETE_WINDOW", self.on_closing)
    
    def create_widgets(self):
        self.frame1 = ttk.Frame(self)
        self.frame1.grid(column=0, row=0, sticky=(tk.W, tk.E), padx=10, pady=10)
        self.button1 = ttk.Button(self.frame1, text="Button1", command=self.command1)
        self.button1.grid()
        
    def command1(self):
        pass
    
    def on_closing(self):
        # ここに終了時の処理を書く
        self.master.destroy()
    
if __name__ == "__main__":
    window = tk.Tk()
    app = SampleWindow(master=window)
    app.mainloop()
~~~

`master.destroy()`を書かないとフォームが閉じられないので注意。

## 配置

全てのウィジェットは配置されなければならない。

配置するためのメソッドは

* `.pack()`
* `.grid()`
* `.place()`

の3種類ある。昔のやり方だと`pack()`が主流だったらしい。今は`grid()`が推奨されている。

参考：https://tkdocs.com/tutorial/concepts.html#geometry

`.grid()`の引数として渡せるパラメーターは以下を参考

参考：https://cercopes-z.com/Python/stdlib-tkinter-widget-methods-py.html#method-grid

