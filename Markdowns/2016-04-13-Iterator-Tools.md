
### Python 迭代器工具

[0x01](https://github.com/rainyear/pytips/blob/master/Markdowns/2016-03-07-iterator-and-generator.md) 介绍了迭代器的概念，即定义了 `__iter__()` 和 `__next__()` 方法的对象，或者通过 `yield` 简化定义的“可迭代对象”，而在一些函数式编程语言（见 [0x02 Python 中的函数式编程](https://github.com/rainyear/pytips/blob/master/Markdowns/2016-03-08-Functional-Programming-in-Python.md)）中，类似的迭代器常被用于产生特定格式的列表（或序列），这时的迭代器更像是一种数据结构而非函数（当然在一些函数式编程语言中，这两者并无本质差异）。Python 借鉴了 `APL, Haskell, and SML` 中的某些迭代器的构造方法，并在 [`itertools`](https://docs.python.org/3/library/itertools.html) 中实现（该模块是通过 C 实现，源代码：[/Modules/itertoolsmodule.c](https://github.com/rainyear/cpython/blob/master/Modules%2Fitertoolsmodule.c)）。

`itertools` 模块提供了如下三类迭代器构建工具：

1. 无限迭代
2. 整合两序列迭代
3. 组合生成器

#### 1. 无限迭代

所谓**无限（infinite）**是指如果你通过 `for...in...` 的语法对其进行迭代，将陷入无限循环，包括：

1. `count(start, [step])`
2. `cycle(p)`
3. `repeat(elem [,n])`

从名字大概可以猜出它们的用法，既然说是无限迭代，我们自然不会想要将其所有元素依次迭代取出，而通常是结合 `map`/`zip` 等方法，将其作为一个取之不尽的数据仓库，与有限长度的可迭代对象进行组合操作：


```python
from itertools import cycle, count, repeat
print(count.__doc__)
```

    count(start=0, step=1) --> count object
    
    Return a count object whose .__next__() method returns consecutive values.
    Equivalent to:
    
        def count(firstval=0, step=1):
            x = firstval
            while 1:
                yield x
                x += step
    



```python
counter = count()
print(next(counter))
print(next(counter))
print(list(map(lambda x, y: x+y, range(10), counter)))

odd_counter = map(lambda x: 'Odd#{}'.format(x), count(1, 2))
print(next(odd_counter))
print(next(odd_counter))
```

    0
    1
    [2, 4, 6, 8, 10, 12, 14, 16, 18, 20]
    Odd#1
    Odd#3



```python
print(cycle.__doc__)
```

    cycle(iterable) --> cycle object
    
    Return elements from the iterable until it is exhausted.
    Then repeat the sequence indefinitely.



```python
cyc = cycle(range(5))
print(list(zip(range(6), cyc)))
print(next(cyc))
print(next(cyc))
```

    [(0, 0), (1, 1), (2, 2), (3, 3), (4, 4), (5, 0)]
    1
    2



```python
print(repeat.__doc__)
```

    repeat(object [,times]) -> create an iterator which returns the object
    for the specified number of times.  If not specified, returns the object
    endlessly.



```python
print(list(repeat('Py', 3)))
rep = repeat('p')
print(list(zip(rep, 'y'*3)))
```

    ['Py', 'Py', 'Py']
    [('p', 'y'), ('p', 'y'), ('p', 'y')]


#### 2. 整合两序列迭代

所谓整合两序列，是指**以两个有限序列为输入，将其整合操作之后返回为一个迭代器**，最为常见的 `zip` 函数就属于这一类别，只不过 `zip` 是内置函数。这一类别完整的方法包括：

1. `accumulate()`
2. `chain()`/`chain.from_iterable()`
3. `compress()`
4. `dropwhile()`/`filterfalse()`/`takewhile()`
5. `groupby()`
6. `islice()`
7. `starmap()`
8. `tee()`
9. `zip_longest()`

这里就不对所有的方法一一举例说明了，如果想要知道某个方法的用法，基本通过 `print(method.__doc__)` 就可以了解，毕竟 `itertools` 模块只是提供了一种快捷方式，并没有隐含什么深奥的算法。这里只对下面几个我觉得比较有趣的方法进行举例说明。


```python
from itertools import cycle, compress, islice, takewhile, count

# 这三个方法（如果使用恰当）可以限定无限迭代
# print(compress.__doc__)
print(list(compress(cycle('PY'), [1, 0, 1, 0])))

# 像操作列表 l[start:stop:step] 一样操作其它序列
# print(islice.__doc__)
print(list(islice(cycle('PY'), 0, 2)))

# 限制版的 filter
# print(takewhile.__doc__)
print(list(takewhile(lambda x: x < 5, count())))
```

    ['P', 'P']
    ['P', 'Y']
    [0, 1, 2, 3, 4]



```python
from itertools import groupby
from operator import itemgetter
print(groupby.__doc__)

for k, g in groupby('AABBC'):
    print(k, list(g))
db = [dict(name='python', script=True),
      dict(name='c', script=False),
      dict(name='c++', script=False),
      dict(name='ruby', script=True)]
keyfunc = itemgetter('script')

db2 = sorted(db, key=keyfunc) # sorted by `script'
for isScript, langs in groupby(db2, keyfunc):
    print(', '.join(map(itemgetter('name'), langs)))
```

    groupby(iterable[, keyfunc]) -> create an iterator which returns
    (key, sub-iterator) grouped by each value of key(value).
    
    A ['A', 'A']
    B ['B', 'B']
    C ['C']
    c, c++
    python, ruby



```python
from itertools import zip_longest

# 内置函数 zip 以较短序列为基准进行合并，
# zip_longest 则以最长序列为基准，并提供补足参数 fillvalue
# Python 2.7 中名为 izip_longest

print(list(zip_longest('ABCD', '123', fillvalue=0)))
```

    [('A', '1'), ('B', '2'), ('C', '3'), ('D', 0)]


#### 3. 组合生成器

关于生成器的排列组合：

1. `product(*iterables, repeat=1)`：两输入序列的笛卡尔乘积
2. `permutations(iterable, r=None)`：对输入序列的完全排列组合
3. `combinations(iterable, r)`：有序版的排列组合
4. `combinations_with_replacement(iterable, r)`：有序版的笛卡尔乘积


```python
from itertools import product, permutations, combinations, combinations_with_replacement
print(list(product(range(2), range(2))))
print(list(product('AB', repeat=2)))
```

    [(0, 0), (0, 1), (1, 0), (1, 1)]
    [('A', 'A'), ('A', 'B'), ('B', 'A'), ('B', 'B')]



```python
print(list(combinations_with_replacement('AB', 2)))
```

    [('A', 'A'), ('A', 'B'), ('B', 'B')]



```python
# 赛马问题：4匹马前2名的排列组合（A^4_2）
print(list(permutations('ABCDE', 2)))
```

    [('A', 'B'), ('A', 'C'), ('A', 'D'), ('A', 'E'), ('B', 'A'), ('B', 'C'), ('B', 'D'), ('B', 'E'), ('C', 'A'), ('C', 'B'), ('C', 'D'), ('C', 'E'), ('D', 'A'), ('D', 'B'), ('D', 'C'), ('D', 'E'), ('E', 'A'), ('E', 'B'), ('E', 'C'), ('E', 'D')]



```python
# 彩球问题：4种颜色的球任意抽出2个的颜色组合（C^4_2）
print(list(combinations('ABCD', 2)))
```

    [('A', 'B'), ('A', 'C'), ('A', 'D'), ('B', 'C'), ('B', 'D'), ('C', 'D')]


### 总结

迭代器工具在产生数据的时候将会显得非常便捷、高效，掌握了这些基本的方法之后，通过简单的组合就可以获得更多迭代器工具。
