
### Python 内置排序方法

Python 提供两种内置排序方法，一个是只针对 `List` 的原地（in-place）排序方法 `list.sort()`，另一个是针对所有可迭代对象的非原地排序方法 `sorted()`。

所谓原地排序是指会立即改变被排序的列表对象，就像 `append()`/`pop()` 等方法一样：


```python
from random import randrange
lst = [randrange(1, 100) for _ in range(10)]
print(lst)
lst.sort()

print(lst)
```

    [57, 81, 32, 74, 12, 89, 76, 21, 75, 6]
    [6, 12, 21, 32, 57, 74, 75, 76, 81, 89]


`sorted()` 不限于列表，而且会生成并返回一个新的排序后的**列表**，原有对象不受影响：


```python
lst = [randrange(1, 100) for _ in range(10)]
tup = tuple(lst)

print(sorted(tup)) # return List
print(tup)
```

    [11, 36, 39, 41, 48, 48, 50, 76, 79, 99]
    (11, 41, 79, 48, 48, 99, 39, 76, 36, 50)


虽然不是原地排序，但如果是传入生成器，还是会被循环掉的：


```python
tup = (randrange(1, 100) for _ in range(10))
print(sorted(tup))
for i in tup:
    print(i)
```

    [5, 12, 15, 21, 57, 69, 73, 83, 90, 95]


### Key

对简单的迭代对象进行排序只需要逐次提取元素进行比较即可，如果想要对元素进行一些操作再进行比较，可以通过 `key` 参数指定一个取值函数。这里的 `key` 用法很像 [0x02 函数式编程](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-08-Functional-Programming-in-Python.ipynb)提到的 `map`/`filter` 所接受的函数，不同之处在于这里的 `key` 函数只是在排序比较前对元素进行处理，并不会改变元素原本的值，例如我们对一组整数**按照（`key` 可以理解为`按照`的意思）**绝对值进行排序：


```python
lst = [randrange(-10, 10) for _ in range(10)]
print(lst)
print(sorted(lst, key=abs))
```

    [0, 7, 0, -10, 3, 7, -9, -10, -7, -10]
    [0, 0, 3, 7, 7, -7, -9, -10, -10, -10]


或者，当迭代对象的元素较为复杂时，可以只**按照**其中的某些属性进行排序：


```python
lst = list(zip("hello world hail python".split(), [randrange(1, 10) for _ in range(4)]))
print(lst)
print(sorted(lst, key=lambda item: item[1]))
```

    [('hello', 3), ('world', 3), ('hail', 9), ('python', 9)]
    [('hello', 3), ('world', 3), ('hail', 9), ('python', 9)]


Python 的 `operator` 标准库提供了一些操作符相关的方法，可以更方便地获取元素的属性：


```python
from operator import itemgetter, attrgetter

print(lst)
print(sorted(lst, key=itemgetter(1)))

# 一切都只是函数
fitemgetter = lambda ind: lambda item: item[ind]
print(sorted(lst, key=fitemgetter(1)))

class P(object):
    def __init__(self, w, n):
        self.w = w
        self.n = n
    def __repr__(self):
        return "{}=>{}".format(self.w, self.n)
ps = [P(i[0], i[1]) for i in lst]

print(sorted(ps, key=attrgetter('n')))
```

    [('hello', 3), ('world', 3), ('hail', 9), ('python', 9)]
    [('hello', 3), ('world', 3), ('hail', 9), ('python', 9)]
    [('hello', 3), ('world', 3), ('hail', 9), ('python', 9)]
    [hello=>3, world=>3, hail=>9, python=>9]


经过 `key` 处理之后会通过 `<` 符号对两个元素进行比较，在 Python 2.7 的版本中，`sorted()` 还可以接收另外一个参数 `cmp`，用来接管 `<` 的比较过程。但是在 Python 3.5 中已经全面摒弃了这一做法，包括 `sorted()` 中的 `cmp` 参数和对象中的 `__cmp__` 比较操作，只有在需要向后兼容的时候才可能在 Python 3.5 用到这一功能，其替换的方法为：


```python
from functools import cmp_to_key as new_cmp_to_key

# new_cmp_to_key works like this
def cmp_to_key(mycmp):
    'Convert a cmp= function into a key= function'
    class K:
        def __init__(self, obj, *args):
            self.obj = obj
        def __lt__(self, other):
            return mycmp(self.obj, other.obj) < 0
    return K
def reverse_cmp(x, y):
    return y[1] - x[1]
sorted(lst, key=cmp_to_key(reverse_cmp))

```




    [('hail', 9), ('python', 9), ('hello', 3), ('world', 3)]



如果想要按照递减排序，只需要设定参数 `reverse = True` 即可。
