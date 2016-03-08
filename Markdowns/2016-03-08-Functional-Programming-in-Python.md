
### Python 中的函数式编程

> 函数式编程（英语：functional programming）或称函数程序设计，又称泛函编程，是一种编程范型，它将电脑运算视为数学上的函数计算，并且避免使用程序状态以及易变对象。函数编程语言最重要的基础是λ演算（lambda calculus）。而且λ演算的函数可以接受函数当作输入（引数）和输出（传出值）。(维基百科：函数式编程)

所谓编程范式（Programming paradigm）是指编程风格、方法或模式，比如面向过程编程（C语言）、面向对象编程（C++）、面向函数式编程（Haskell），并不是说某种编程语言一定属于某种范式，例如 Python 就是多范式编程语言。

#### 函数式编程

函数式编程具有以下特点：

1. 避免状态变量
2. 函数也是变量（一等公民，First-Class Citizen）
3. 高阶函数
4. 面向问题描述而不是面向问题解决步骤

值得一提的是，函数式编程的这些特点在实践过程中可能并不是那么 Pythonic，甚至与**[0x00](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-06-The-Zen-of-Python.ipynb)**中提到的 The Zen of Python 相悖。例如函数式编程面向问题描述的特点可能让你更快地写出更简洁的代码，但可读性却也大打折扣（可参考这一段[Haskell代码](https://gist.github.com/rainyear/94b5d9a865601f075719)）。不过，虽然 Pythonic 很重要但并不是唯一的准则，_The Choice Is Yours_。

#### `map(function, iterable, ...)`/`filter(function, iterable)`


```python
# map 函数的模拟实现
def myMap(func, iterable):
    for arg in iterable:
        yield func(arg)

names = ["ana", "bob", "dogge"]

print(map(lambda x: x.capitalize(), names)) # Python 2.7 中直接返回列表
for name in myMap(lambda x: x.capitalize(), names):
    print(name)
```

    <map object at 0x11185c9b0>
    Ana
    Bob
    Dogge



```python
# filter 函数的模拟实现
def myFilter(func, iterable):
    for arg in iterable:
        if func(arg):
            yield arg
            
print(filter(lambda x: x % 2 == 0, range(10))) # Python 2.7 中直接返回列表
for i in myFilter(lambda x: x % 2 == 0, range(10)):
    print(i)
```

    <filter object at 0x11185cbe0>
    0
    2
    4
    6
    8


#### `functools.reduce(function, iterable[, initializer])`

Python 3.5 中`reduce` 被降格到标准库`functools`，`reduce` 也是遍历可迭代对象元素作为第一个函数的参数，并将结果累计：


```python
from functools import reduce

print(reduce(lambda a, b: a*b, range(1,5)))
```

    24


#### `functools.partial(func, *args, **keywords)`

偏应用函数（Partial Application）让我们可以固定函数的某些参数：


```python
from functools import partial

add = lambda a, b: a + b
add1024 = partial(add, 1024)

add1024(1)
```




    1025



这里简单介绍了一些常用函数式编程的方法和概念，实际上要传达的一个最重要的观念就是**函数本身也可以作为变量被返回、传递给高阶函数**，这使得我们可以更灵活地运用函数解决问题。但是这并不意味着一定要使用上面这些方法来简化代码，例如更 Pythonic 的方法推荐尽可能使用 List Comprehension 替代`map`/`filter`（关于 List Comprehension 后面会再单独介绍）。如果一定想要用函数式编程的方法来写 Python，也可以尝试[Fn.py](https://github.com/kachayev/fn.py)，或者，试试 [Haskell](https://www.haskell.org/)。

#### 参考

1. [维基百科：函数式编程](https://zh.wikipedia.org/wiki/%E5%87%BD%E6%95%B8%E7%A8%8B%E5%BC%8F%E8%AA%9E%E8%A8%80)
2. [byvoid：APIO讲稿——函数式编程](http://byvoid.github.io/slides/apio-fp/index.html)
