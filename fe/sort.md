# ソートアルゴリズム

言語はPython。ただし、アルゴリズム部分ではなるべくPythonに依存しないように書く。（変数の中身を交換する処理を除く）

## 乱数列を取得

~~~python
import random
~~~

~~~python
def getlist(len):
    return random.sample(range(len), k=len)
~~~

~~~python
def getrandintlist(len):
    return [random.randint(0, len-1) for i in range(len)]
~~~

## 評価

~~~python
import time
~~~

~~~python
def eval_sort(arr):
    failed = False
    for i in range(len(arr)-1):
        if arr[i] > arr[i+1]:
            failed = True
            break
    if failed:
        print('Failed sorting...')
    else:
        print('Sorted!')
~~~

~~~python
def eval_time(arr, sort_algorithm):
    start = time.time()
    sort_algorithm(arr)
    print('elapsed {} sec'.format(time.time() - start))
~~~

## ソートアルゴリズム

~~~python
def bubble(arr):
    n = len(arr)
    for i in range(n-1, 0, -1): # bubbleはゴールをイテレート
        for j in range(i):
            if arr[j] > arr[j+1]:
                arr[j], arr[j+1] = arr[j+1], arr[j]
~~~

~~~python
def selection(arr):
    n = len(arr)
    for i in range(n-1): # selectはスタートをイテレート
        min = i
        for j in range(i+1, n):
            if arr[j] < arr[min]:
                min = j
        arr[i], arr[min] = arr[min], arr[i]
~~~

~~~python
def insertion(arr):
    n = len(arr)
    for i in range(1, n): # insertionはスタートをイテレート
        tmp = arr[i]
        idx = i
        while idx >= 1 and arr[idx-1] > tmp: # ただしスタートから後ろに戻る
            arr[idx] = arr[idx-1]
            idx -= 1
        arr[idx] = tmp
~~~

~~~python
def h_sort(arr, h, k=0):
    n = len(arr)
    for i in range(k+h, n, h):
        tmp = arr[i]
        idx = i
        while idx >= h and arr[idx-h] > tmp:
            arr[idx] = arr[idx-h]
            idx -= h
        arr[idx] = tmp
~~~

~~~python
def shell(arr):
    n = len(arr)
    while (h < n // 9):
        h = h * 3 + 1
    while h > 0:
        for i in range(h, n):
            tmp = arr[i]
            idx = i
            while idx >= h and arr[idx-h] > tmp:
                arr[idx] = arr[idx-h]
                idx -= h
            arr[idx] = tmp
        h //= 3
~~~

~~~python
def qsort(arr, l, r):
    pl = l
    pr = r
    x = arr[(l + r) // 2]
    while pl <= pr:
        while arr[pl] < x: pl += 1
        while arr[pr] > x: pr -= 1
        if pl <= pr:
            arr[pl], arr[pr] = arr[pr], arr[pl]
            pl += 1
            pr -= 1
    if l < pr: qsort(arr, l, pr)
    if pl < r: qsort(arr, pl, r)
        
def quick(arr):
    qsort(arr, 0, len(arr)-1)
~~~

~~~python
def msort(arr, l, r, buff):
    if l < r:
        c = (l + r) // 2
        msort(arr, l, c, buff)
        msort(arr, c+1, r, buff)
        
        blen = bidx = 0
        i = j = l
        while i <= c:
            buff[blen] = arr[i]
            blen += 1
            i += 1
        while i <= r and bidx < blen:
            if buff[bidx] <= arr[i]:
                arr[j] = buff[bidx]
                bidx += 1
            else:
                arr[j] = arr[i]
                i += 1
            j += 1
        while bidx < blen:
            arr[j] = buff[bidx]
            j += 1
            bidx += 1
            
def merge(arr):
    n = len(arr)
    buff = [None] * n
    msort(arr, 0, n-1, buff)
    del buff
~~~

