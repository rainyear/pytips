
## Python 上下文管理器

Python 2.5 引入了 `with` 语句（[PEP 343](https://www.python.org/dev/peps/pep-0343/)）与上下文管理器类型（[Context Manager Types](https://docs.python.org/3/library/stdtypes.html#context-manager-types)），其主要作用包括：

> 保存、重置各种全局状态，锁住或解锁资源，关闭打开的文件等。[With Statement Context Managers](https://docs.python.org/3/reference/datamodel.html#with-statement-context-managers)

一种最普遍的用法是对文件的操作：


```python
with open("utf8.txt", "r") as f:
    print(f.read())
```

    你好，世界！
    


上面的例子也可以用 `try...finally...` 实现，它们的效果是相同（或者说上下文管理器就是封装、简化了错误捕捉的过程）：


```python
try:
    f = open("utf8.txt", "r")
    print(f.read())
finally:
    f.close()
```

    你好，世界！
    


除了文件对象之外，我们也可以自己创建上下文管理器，与 [0x01](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-07-iterator-and-generator.ipynb) 中介绍的迭代器类似，只要定义了 `__enter__()` 和 `__exit__()` 方法就成为了上下文管理器类型。`with` 语句的执行过程如下：

1. 执行 `with` 后的语句获取上下文管理器，例如 `open('utf8.txt', 'r')` 就是返回一个 `file object`；
2. 加载 `__exit__()` 方法备用；
3. 执行 `__enter__()`，该方法的返回值将传递给 `as` 后的变量（如果有的话）；
4. 执行 `with` 语法块的子句；
5. 执行 `__exit__()` 方法，如果 `with` 语法块子句中出现异常，将会传递 `type, value, traceback` 给 `__exit__()`，否则将默认为 `None`；如果 `__exit__()` 方法返回 `False`，将会抛出异常给外层处理；如果返回 `True`，则忽略异常。

了解了 `with` 语句的执行过程，我们可以编写自己的上下文管理器。假设我们需要一个引用计数器，而出于某些特殊的原因需要多个计数器共享全局状态并且可以相互影响，而且在计数器使用完毕之后需要恢复初始的全局状态：


```python
_G = {"counter": 99, "user": "admin"}

class Refs():
    def __init__(self, name = None):
        self.name = name
        self._G = _G
        self.init = self._G['counter']
    def __enter__(self):
        return self
    def __exit__(self, *args):
        self._G["counter"] = self.init
        return False
    def acc(self, n = 1):
        self._G["counter"] += n
    def dec(self, n = 1):
        self._G["counter"] -= n
    def __str__(self):
        return "COUNTER #{name}: {counter}".format(**self._G, name=self.name)
        
with Refs("ref1") as ref1, Refs("ref2") as ref2: # Python 3.1 加入了多个并列上下文管理器
    for _ in range(3):
        ref1.dec()
        print(ref1)
        ref2.acc(2)
        print(ref2)
print(_G)
```

    COUNTER #ref1: 98
    COUNTER #ref2: 100
    COUNTER #ref1: 99
    COUNTER #ref2: 101
    COUNTER #ref1: 100
    COUNTER #ref2: 102
    {'user': 'admin', 'counter': 99}


上面的例子很别扭但是可以很好地说明 `with` 语句的执行顺序，只是每次定义两个方法看起来并不是很简洁，一如既往地，Python 提供了 `@contextlib.contextmanager` + `generator` 的方式来简化这一过程（正如 [0x01](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-07-iterator-and-generator.ipynb) 中 `yield` 简化迭代器一样）：


```python
from contextlib import contextmanager as cm
_G = {"counter": 99, "user": "admin"}

@cm
def ref():
    counter = _G["counter"]
    yield _G
    _G["counter"] = counter

with ref() as r1, ref() as r2:
    for _ in range(3):
        r1["counter"] -= 1
        print("COUNTER #ref1: {}".format(_G["counter"]))
        r2["counter"] += 2
        print("COUNTER #ref2: {}".format(_G["counter"]))
print("*"*20)
print(_G)
```

    COUNTER #ref1: 98
    COUNTER #ref2: 100
    COUNTER #ref1: 99
    COUNTER #ref2: 101
    COUNTER #ref1: 100
    COUNTER #ref2: 102
    ********************
    {'user': 'admin', 'counter': 99}


这里对生成器的要求是必须只能返回一个值（只有一次 `yield`），返回的值相当于 `__enter__()` 的返回值；而 `yield` 后的语句相当于 `__exit__()`。

生成器的写法更简洁，适合快速生成一个简单的上下文管理器。

除了上面两种方式，Python 3.2 中新增了 `contextlib.ContextDecorator`，可以允许我们自己在 `class` 层面定义新的”上下文管理修饰器“，有兴趣可以到[官方文档查看](https://docs.python.org/3/library/contextlib.html#contextlib.ContextDecorator)。至少在我目前看来好像并没有带来更多方便（除了可以省掉一层缩进之外:(）。

上下文管理器的概念与修饰器有很多相似之处，但是要记住的是 `with` 语句的目的是为了更优雅地收拾残局而不是替代 `try...finally...`，毕竟在 [The Zen of Python](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-06-The-Zen-of-Python.ipynb) 中，

> Explicit is better than implicit.

比

> Simple is better than complex.

更重要:P。
