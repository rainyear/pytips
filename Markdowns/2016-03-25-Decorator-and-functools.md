
## Python 修饰器与 `functools`

Python 的修饰器是一种语法糖（Syntactic Sugar），也就是说：

```python
@decorator
@wrap
def func():
    pass
```

是下面语法的一种简写：

```python
def func():
    pass
func = decorator(wrap(func))
```

关于修饰器的两个主要问题：

1. 修饰器用来修饰谁
2. 谁可以作为修饰器

### 修饰函数

修饰器最常见的用法是修饰新定义的函数，在 [0x0d 上下文管理器](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-23-With-Context-Manager.ipynb)中提到上下文管理器主要是为了**更优雅地完成善后工作**，而修饰器通常用于扩展函数的行为或属性：


```python
def log(func):
    def wraper():
        print("INFO: Starting {}".format(func.__name__))
        func()
        print("INFO: Finishing {}".format(func.__name__))
    return wraper

@log
def run():
    print("Running run...")
run()
```

    INFO: Starting run
    Running run...
    INFO: Finishing run


### 修饰类

除了修饰函数之外，Python 3.0 之后增加了对新定义类的修饰（[PEP 3129](https://www.python.org/dev/peps/pep-3129/)），但是对于类别属性的修改可以通过 [`Metaclasses`](https://www.python.org/doc/essays/metaclasses/) 或继承来实现，而新增加的类别修饰器更多是出于 [Jython](https://mail.python.org/pipermail/python-dev/2006-March/062942.html) 以及 [IronPython](http://lists.ironpython.com/pipermail/users-ironpython.com/2006-March/002007.html) 的考虑，但其语法还是很一致的：


```python
from time import sleep, time
def timer(Cls):
    def wraper():
        s = time()
        obj = Cls()
        e = time()
        print("Cost {:.3f}s to init.".format(e - s))
        return obj
    return wraper
@timer
class Obj:
    def __init__(self):
        print("Hello")
        sleep(3)
        print("Obj")
o = Obj()
```

    Hello
    Obj
    Cost 3.005s to init.


### 类作为修饰器

上面两个例子都是以函数作为修饰器，因为函数才可以被调用（callable） `decorator(wrap(func))`。除了函数之外，我们也可以定义可被调用的类，只要添加 `__call__` 方法即可：


```python
class HTML(object):
    """
        Baking HTML Tags!
    """
    def __init__(self, tag="p"):
        print("LOG: Baking Tag <{}>!".format(tag))
        self.tag = tag
    def __call__(self, func):
        return lambda: "<{0}>{1}</{0}>".format(self.tag, func(), self.tag)

@HTML("html")
@HTML("body")
@HTML("div")
def body():
    return "Hello"

print(body())
```

    LOG: Baking Tag <html>!
    LOG: Baking Tag <body>!
    LOG: Baking Tag <div>!
    <html><body><div>Hello</div></body></html>


### 传递参数

在实际使用过程中，我们可能需要向修饰器传递参数，也有可能需要向被修饰的函数（或类）传递参数。按照语法约定，只要修饰器 `@decorator` 中的 `decorator` 是可调用即可，`decorator(123)` 如果返回一个新的可调用函数，那么也是合理的，上面的 `@HTML('html')` 即是一例，下面再以 [flask](https://github.com/mitsuhiko/flask/blob/master/flask%2Fapp.py) 的路由修饰器为例说明如何传递参数给修饰器：


```python
RULES = {}
def route(rule):
    def decorator(hand):
        RULES.update({rule: hand})
        return hand
    return decorator
@route("/")
def index():
    print("Hello world!")

def home():
    print("Welcome Home!")
home = route("/home")(home)

index()
home()
print(RULES)
```

    Hello world!
    Welcome Home!
    {'/': <function index at 0x10706f730>, '/home': <function home at 0x10706f8c8>}


向被修饰的函数传递参数，要看我们的修饰器是如何作用的，如果像上面这个例子一样未执行被修饰函数只是将其原模原样地返回，则不需要任何处理（这就把函数当做普通的值一样看待即可）：


```python
@route("/login")
def login(user = "user", pwd = "pwd"):
    print("DB.findOne({{{}, {}}})".format(user, pwd))
login("hail", "python")
```

    DB.findOne({hail, python})


如果需要在修饰器内执行，则需要稍微变动一下：


```python
def log(f):
    def wraper(*args, **kargs):
        print("INFO: Start Logging")
        f(*args, **kargs)
        print("INFO: Finish Logging")
    return wraper

@log
def run(hello = "world"):
    print("Hello {}".format(hello))
run("Python")
```

    INFO: Start Logging
    Hello Python
    INFO: Finish Logging


### functools

由于修饰器将函数（或类）进行包装之后重新返回：`func = decorator(func)`，那么有可能改变原本函数（或类）的一些信息，以上面的 `HTML` 修饰器为例：


```python
@HTML("body")
def body():
    """
        return body content
    """
    return "Hello, body!"
print(body.__name__)
print(body.__doc__)
```

    LOG: Baking Tag <body>!
    <lambda>
    None


因为 `body = HTML("body")(body)` ，而 `HTML("body").__call__()` 返回的是一个 `lambda` 函数，因此 `body` 已经被替换成了 `lambda`，虽然都是可执行的函数，但原来定义的 `body` 中的一些属性，例如 `__doc__`/`__name__`/`__module__` 都被替换了（在本例中`__module__`没变因为都在同一个文件中）。为了解决这一问题 Python 提供了 [`functools`](https://docs.python.org/3.5/library/functools.html) 标准库，其中包括了 `update_wrapper` 和 `wraps` 两个方法（[源码](https://hg.python.org/cpython/file/3.5/Lib/functools.py)）。其中 `update_wrapper` 就是用来将原来函数的信息赋值给修饰器中返回的函数：


```python
from functools import update_wrapper
"""
functools.update_wrapper(wrapper, wrapped[, assigned][, updated])
"""


class HTML(object):
    """
        Baking HTML Tags!
    """
    def __init__(self, tag="p"):
        print("LOG: Baking Tag <{}>!".format(tag))
        self.tag = tag
    def __call__(self, func):
        wraper = lambda: "<{0}>{1}</{0}>".format(self.tag, func(), self.tag)
        update_wrapper(wraper, func)
        return wraper
@HTML("body")
def body():
    """
        return body content!
    """
    return "Hello, body!"
print(body.__name__)
print(body.__doc__)
```

    LOG: Baking Tag <body>!
    body
    
            return body content!
        


有趣的是 `update_wrapper` 的用法本身就很像是修饰器，因此 `functools.wraps` 就利用 `functools.partial`（还记得函数式编程中的偏应用吧！）将其变成一个修饰器：


```python
from functools import update_wrapper, partial

def my_wraps(wrapped):
    return partial(update_wrapper, wrapped=wrapped)

def log(func):
    @my_wraps(func)
    def wraper():
        print("INFO: Starting {}".format(func.__name__))
        func()
        print("INFO: Finishing {}".format(func.__name__))
    return wraper

@log
def run():
    """
    Docs' of run
    """
    print("Running run...")
print(run.__name__)
print(run.__doc__)
```

    run
    
        Docs' of run
        


### 参考

1. [Python修饰器的函数式编程](http://coolshell.cn/articles/11265.html)
