# Make 10

~~~python
import itertools
~~~

~~~python
def is_formula(f):
    if type(f[0]).__name__ != 'int':
        return False
    if type(f[1]).__name__ != 'int':
        return False
    if type(f[-1]).__name__ != 'str':
        return False
~~~

~~~python
def calc(f):
    if is_formula(f) == False:
        return None
    stack = []
    for element in f:
        if type(element).__name__ == 'int':
            stack.append(element)
        else:
            if len(stack) < 2:
                return None
            tmp2, tmp1 = stack.pop(), stack.pop()
            if element == '+':
                stack.append(tmp1 + tmp2)
            elif element == '-':
                stack.append(tmp1 - tmp2)
            elif element == '*':
                stack.append(tmp1 * tmp2)
            elif element == '/' and tmp2 != 0:
                stack.append(tmp1 / tmp2)
            else:
                return None
    if len(stack) == 1:
        return stack.pop()
    else:
        return None
~~~

~~~python
def makeexp(f):
	ret = [str(element) if type(element).__name__ == 'int' else element for element in f]
    ops = ['+', '-', '*', '/']
    i = 0
    while i < len(ret):
        if ret[i] in ops:
            ret.insert(i-2, f'( {ret.pop(i-2)} {ret.pop(i-1)} {ret.pop(i-2)} )')
            i -= 1
        else
        	i += 1
    return ret.pop()
~~~

~~~python
def make10(num):
    formulas = [num + list(op) for op in itertools.combinations_with_replacement(['+', '-', '*', '/'], 3)]
    ret = []
	for f in formulas:
        seen = []
        ret += [list(x) for x in itertools.permutations(f) if calc(x) == 10 and x not in seen and not seen.append(x)]
    for r in ret:
        print(f'{makeexp(r)} = 10')
~~~

