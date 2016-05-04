
## Python 类与元类的深度挖掘 II

上一篇解决了通过调用**类对象**生成**实例对象**过程中可能遇到的命名空间相关的一些问题，这次我们向上回溯一层，看看类对象本身是如何产生的。

我们知道 `type()` 方法可以查看一个对象的类型，或者说判断这个对象是由那个类产生的：


```python
print(type(12))
print(type('python'))
```

    <class 'int'>
    <class 'str'>



```python
class A:
    pass
print(type(A))
```

    <class 'type'>


通过这段代码可以看出，类对象 `A` 是由`type()` 产生的，也就是说 `type` 也可以用来产生新的对象，而且产生的是**类对象**，因此它是所有类对象的类：


```python
print(type.__doc__)
```

    type(object_or_name, bases, dict)
    type(object) -> the object's type
    type(name, bases, dict) -> a new type


`class` 定义类的语法实际上转化为 `type(name, bases, dict)`，其中 `name` 参数为类的名字，`bases` 为继承父类的元组，`dict` 为类的属性和方法：


```python
class A:
    pass
# 实际上等于
B = type('A', (), {})

print(A.__name__ == B.__name__)
```

    True


理论上说这就是元类的意义，但从实际的角度出发显然使用 `class` 语法更方便、合理，而元类的实际意义则是通过继承 `type` 类来构造一个新的元类，并进行特定的操作以产生具有特定行为的类对象。这样看来它的本质与普通的类对象没有差异，只不过继承的是 `type` 类。

在生成实例时是通过调用 `__init__` 方法进行初始化的，而实际上在此之前会先调用 `__new__` 方法用于**创建**实例，再通过 `__init__` 初始化，就好像 `__new__` 负责声明变量，而 `__init__` 负责对声明的变量进行初始化一样。这里有一个规则是 `__new__(cls,)` 的返回值必须是 `cls` 参数的实例，否则 `__init__` 将不会触发，例如在 `enum.Enum` 的定义中，由于枚举类型是单例模式，因此在定义 `__new__` 的时候没有返回其实例，也就不会进行初始化：


```python
class Enum:
    def __new__(cls, value):
        print(cls, value)
        return value
    def __init__(self):
        print("Will not be called!")
e = Enum(1)
```

    <class '__main__.Enum'> 1


通常情况下自己定义 `__new__` 需要通过调用父类的 `__new__` 方法创建一个 `cls` 的实例，同样在定义元类的时候则是调用上面提到的 `type` 的用法（因为元类继承自 `type`）：


```python
class MetaEnum(type):
    def __new__(metaclass, name, base, attrs):
        print("Metaclass: {}\nName: {}\nParents: {}\nAttributes: {}".format(metaclass, name, base, attrs))
        return super().__new__(metaclass, name, base, attrs)
```


```python
class Enum(metaclass=MetaEnum):
    # Python 2.7 中定义元类的方法是使用 __metaclass__ 变量
    # [PEP 3115](https://www.python.org/dev/peps/pep-3115/)
    # 将 Python 3.0 以后语法改为 class Cls(metaclass=Meta)
    test = 0
```

    Metaclass: <class '__main__.MetaEnum'>
    Name: Enum
    Parents: ()
    Attributes: {'__qualname__': 'Enum', '__module__': '__main__', 'test': 0}


此时我们再来看 `Enum` 的类，已经不再是 `type` 而是其元类 `MetaEnum`：


```python
type(Enum)
```




    __main__.MetaEnum



除了 `__new__` 方法之外，[PEP 3115](https://www.python.org/dev/peps/pep-3115/) 还定义了 `__prepare__` 属性，用于设定初始化的命名空间（即 `type` 的第 3 个参数），还是以 `enum.Enum` 为例，我们需要限制枚举类型中属性名称不得重复使用，则可以通过元类限制类的行为：


```python
# 定义新的字典类，在赋值新的 dict[k] = v 时
# 检查 k 是否重复
class _EnumDict(dict):
    def __init__(self):
        super().__init__()
        self.members = []
    def __setitem__(self, k, v):
        if k in self.members:
            raise TypeError("Attempted to reuse key: '{}'".format(k))
        else:
            self.members.append(k)
            super().__setitem__(k, v)
            
class MetaEnum(type):
    @classmethod
    def __prepare__(metaclass, cls, bases):
        return _EnumDict()
    def __new__(metaclass, name, base, attrs):
        return super().__new__(metaclass, name, base, attrs)
class Enum(metaclass=MetaEnum):
    pass

class Color(Enum):
    try:
        red = 1
        red = 2
    except TypeError:# 这里没有使用 as err: 的原因是？
        print("TypeError catched")
```

    TypeError catched


Python 中一切皆为对象，所有的对象都是某一类的实例，或是某一元类的实例，`type` 是自己的元类也是自己的实例：

![metaclass](http://qncdn.rainy.im/pytips/metaclass2.jpg)

### 总结

元类在 Python 中属于比较深层的黑魔法，在一般的日常应用中可能并不常用，但理解其背后的原理对于理解 Python 面向对象编程以及一切皆为对象的理念很有帮助；如果你需要对类进行深度改造，至少要知道从何入手。

### 参考

1. [What is a metaclass in Python](http://stackoverflow.com/questions/100003/what-is-a-metaclass-in-python)
