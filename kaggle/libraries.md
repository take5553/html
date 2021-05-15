# Pythonのライブラリの導入

必要なライブラリの導入をまとめておく

[TOC]

## NumPy

ターミナル上で

~~~shell
$ sudo apt install libatlas-base-dev
$ sudo apt install python3-numpy
~~~

Jupyter Notebook上で

~~~python
import numpy as np
x = np.array([1,2,3])
print(x)
~~~

## Matplotlib

仮想環境内で

~~~shell
(.venv) $ python3 -m pip install matplotlib
~~~

Jupyter Notebook上で

~~~python
import numpy as np
import matplotlib.pylab as plt
x = np.arange(0, 6, 0.1)
y = np.sin(x)
plt.plot(x, y)
plt.show()
~~~

