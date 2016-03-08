
### 迭代器与生成器

迭代器（iterator）与生成器（generator）是 Python 中比较常用又很容易混淆的两个概念，今天就把它们梳理一遍，并举一些常用的例子。

**`for` 语句与可迭代对象（iterable object）：**


```python
for i in [1, 2, 3]:
    print(i)
```

    1
    2
    3



```python
obj = {"a": 123, "b": 456}
for k in obj:
    print(k)
```

    b
    a


这些可以用在 `for` 语句进行循环的对象就是**可迭代对象**。除了内置的数据类型（列表、元组、字符串、字典等）可以通过 `for` 语句进行迭代，我们也可以自己创建一个容器，包含一系列元素，可以通过 `for` 语句依次循环取出每一个元素，这种容器就是**迭代器（iterator）**。除了用 `for` 遍历，迭代器还可以通过 `next()` 方法逐一读取下一个元素。要创建一个迭代器有3种方法，其中前两种分别是：

1. 为容器对象添加 `__iter__()` 和 `__next__()` 方法（Python 2.7 中是 `next()`）；`__iter__()` 返回迭代器对象本身 `self`，`__next__()` 则返回每次调用 `next()` 或迭代时的元素；
2. 内置函数 `iter()` 将可迭代对象转化为迭代器


```python
# iter(IterableObject)
ita = iter([1, 2, 3])
print(type(ita))

print(next(ita))
print(next(ita))
print(next(ita))

# Create iterator Object
class Container:
    def __init__(self, start = 0, end = 0):
        self.start = start
        self.end = end
    def __iter__(self):
        print("[LOG] I made this iterator!")
        return self
    def __next__(self):
        print("[LOG] Calling __next__ method!")
        if self.start < self.end:
            i = self.start
            self.start += 1
            return i
        else:
            raise StopIteration()
c = Container(0, 5)
for i in c:
    print(i)
        
```

    <class 'list_iterator'>
    1
    2
    3
    [LOG] I made this iterator!
    [LOG] Calling __next__ method!
    0
    [LOG] Calling __next__ method!
    1
    [LOG] Calling __next__ method!
    2
    [LOG] Calling __next__ method!
    3
    [LOG] Calling __next__ method!
    4
    [LOG] Calling __next__ method!


创建迭代器对象的好处是当序列长度很大时，可以减少内存消耗，因为每次只需要记录一个值即刻（经常看到人们介绍 Python 2.7 的 `range` 函数时，建议当长度太大时用 `xrange` 更快，在 Python 3.5 中已经去除了 `xrange` 只有一个类似迭代器一样的 `range`）。

#### 生成器

前面说到创建迭代器有3种方法，其中第三种就是**生成器（generator）**。生成器通过 `yield` 语句快速生成迭代器，省略了复杂的 `__iter__()` & `__next__()` 方式：


```python
def container(start, end):
    while start < end:
        yield start
        start += 1
c = container(0, 5)
print(type(c))
print(next(c))
next(c)
for i in c:
    print(i)
```

    <class 'generator'>
    0
    2
    3
    4


简单来说，`yield` 语句可以让普通函数变成一个生成器，并且相应的 `__next__()` 方法返回的是 `yield` 后面的值。一种更直观的解释是：程序执行到 `yield` 会返回值并暂停，再次调用 `next()` 时会从上次暂停的地方继续开始执行：


```python
def gen():
    yield 5
    yield "Hello"
    yield "World"
    yield 4
for i in gen():
    print(i)
```

    5
    Hello
    World
    4


Python 3.5 （准确地说应该是 3.3 以后）中为生成器添加了更多特性，包括 `yield from` 以及在暂停的地方传值回生成器的 `send()`等，为了保持简洁这里就不深入介绍了，有兴趣可以阅读[官方文档](https://docs.python.org/3/reference/expressions.html#yieldexpr)说明以及参考链接2。

#### 参考

1. [Iterators & Generators](http://anandology.com/python-practice-book/iterators.html)
2. [How the heck does async/await work in Python 3.5?](http://www.snarky.ca/how-the-heck-does-async-await-work-in-python-3-5)
3. [Python's yield from](http://charlesleifer.com/blog/python-s-yield-from/)
