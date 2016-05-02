
## Python 类与元类的深度挖掘 I

上一篇介绍了 Python 枚举类型的标准库，除了考虑到其实用性，还有一个重要的原因是其实现过程是一个非常好的学习、理解 Python 类与元类的例子。因此接下来两篇就以此为例，深入挖掘 Python 中类与元类背后的机制。

翻开任何一本 Python 教程，你一定可以在某个位置看到下面这两句话：

1. Python 中一切皆为对象（Everything in Python is an object）；
2. Python 是一种面向对象编程（Object Oriented Programming, OOP）的语言。

虽然在上面两句话的语境中，对象（Object）的含义可能稍有不同，但可以肯定的是对象在 Python 中具有非常重要的意义，也是我们接下来将要讨论的所有内容的基础。那么，对象到底是什么？

![object](http://qncdn.rainy.im/pytips/object.png)

### 对象（Object）

> 对象是 Python 中对数据的一种抽象，Python 程序中所有数据都是通过对象或对象之间的关系来表示的。[[ref: Data Model]](https://docs.python.org/2.7/reference/datamodel.html)

港台将 Object 翻译为“物件”，可以将其看作是一个盛有数据的盒子，只不过除了纯粹的数据之外还有其它有用的属性信息，在 Python 中，所有的对象都具有`id`、`type`、`value`三个属性：

```
+---------------+
|               |
| Python Object |
|               |
+------+--------+
| ID   |        |
+---------------+
| Type |        |
+---------------+
| Value|        |
+---------------+
```

其中 `id` 代表内存地址，可以通过内置函数 `id()` 查看，而 `type` 表示对象的类别，不同的类别意味着该对象拥有的属性和方法等，可以通过 `type()` 方法查看：


```python
def who(obj):
    print(id(obj), type(obj))
    
who(1)
who(None)
who(who)
```

    4515088368 <class 'int'>
    4514812344 <class 'NoneType'>
    4542646064 <class 'function'>


对象作为 Python 中的基本单位，可以被创建、命名或删除。Python 中一般不需要手动删除对象，其垃圾回收机制会自动处理不再使用的对象，当然如果需要，也可以使用 `del` 语句删除某个变量；所谓命名则是指给对象贴上一个名字标签，方便使用，也就是声明或赋值变量；接下来我们重点来看如何创建一个对象。对于一些 Python 内置类型的对象，通常可以使用特定的语法生成，例如数字直接使用阿拉伯数字字面量，字符串使用引号 `''`，列表使用 `[]`，字典使用 `{}` ，函数使用 `def` 语法等，这些对象的类型都是 Python 内置的，那我们能不能创建其它类型的对象呢？

### 类与实例

既然说 Python 是面向对象编程语言，也就允许用户自己创建对象，通常使用 `class` 语句，与其它对象不同的是，`class` 定义的对象（称之为**类**）可以用于产生新的对象（称之为**实例**）：


```python
class A:
    pass
a = A()
who(A)
who(a)
```

    140477703944616 <class 'type'>
    4542635424 <class '__main__.A'>


上面的例子中 `A` 是我们创建的一个新的类，而通过调用 `A()` 可以获得一个 `A` 类型的实例对象，我们将其赋值为 `a`，也就是说我们成功创建了一个与所有内置对象类型不同的对象 `a`，它的类型为 `__main__.A`！至此我们可以将 Python 中一切的对象分为两种：

1. 可以用来生成新对象的类，包括内置的 `int`、`str` 以及自己定义的 `A` 等；
2. 由类生成的实例对象，包括内置类型的数字、字符串以及自己定义的类型为 `__main__.A` 的 `a`。

单纯从概念上理解这两种对象没有任何问题，但是这里要讨论的是在实践中不得不考虑的一些细节性问题：

1. 需要一些方便的机制来实现面向对象编程中的继承、重载等特性；
2. 需要一些固定的流程让我们可以在生成实例化对象的过程中执行一些特定的操作；

这两个问题主要关于类的一些特殊的操作，也就是这一篇后面的主要内容。如果再回顾一下开头提到的两句话，你可能会想到，既然类本身也是对象，那它们又是怎样生成的？这就是后一篇将主要讨论的问题：用于生成类对象的类，即元类（Metaclass）。

### `super`, `mro()`

0x00 Python 之禅中提到的最后一条，命名空间（namespace）是个绝妙的理念，类或对象在 Python 中就承担了一部分命名空间的作用。比如说某些特定的方法或属性只有特定类型的对象才有，不同类型对象的属性和方法尽管名字可能相同，但由于隶属不同的命名空间，其值可能完全不同。在实现类的继承与重载等特性时同样需要考虑命名空间的问题，以枚举类型的实现为例，我们需要保证枚举对象的属性名称不能有重复，因此我们需要继承内置的 `dict` 类：


```python
class _EnumDict(dict):
    def __init__(self):
        dict.__init__(self)
        self._member_names = []
    def keys(self):
        keys = dict.keys(self)
        return list(filter(lambda k: k.isupper(), keys))

ed = _EnumDict()
ed['RED'] = 1
ed['red'] = 2
print(ed, ed.keys())
```

    {'RED': 1, 'red': 2} ['RED']


在上面的例子中 `_EnumDict` 重载同时调用了父类 `dict` 的一些方法，上面的写法在语法上是没有错误的，但是如果我们要改变 `_EnumDict` 的父类，不再是继承自 `dict`，则必须手动修改所有方法中 `dict.method(self)` 的调用形式，这样就不是一个好的实践方案了。为了解决这一问题，Python 提供了一个内置函数 `super()`：


```python
print(super.__doc__)
```

    super() -> same as super(__class__, <first argument>)
    super(type) -> unbound super object
    super(type, obj) -> bound super object; requires isinstance(obj, type)
    super(type, type2) -> bound super object; requires issubclass(type2, type)
    Typical use to call a cooperative superclass method:
    class C(B):
        def meth(self, arg):
            super().meth(arg)
    This works for class methods too:
    class C(B):
        @classmethod
        def cmeth(cls, arg):
            super().cmeth(arg)
    


我最初只是把 `super()` 当做指向父类对象的指针，但实际上它可以提供更多功能：给定一个对象及其子类（这里对象要求至少是类对象，而子类可以是实例对象），从该对象父类的命名空间开始搜索对应的方法。

以下面的代码为例：


```python
class A(object):
    def method(self):
        who(self)
        print("A.method")
class B(A):
    def method(self):
        who(self)
        print("B.method")
class C(B):
    def method(self):
        who(self)
        print("C.method")
class D(C):
    def __init__(self):
        super().method()
        super(__class__, self).method()
        
        super(C, self).method() # calling C's parent's method
        super(B, self).method() # calling B's parent's method
        
        super(B, C()).method()  # calling B's parent's method with instance of C
```


```python
d = D()

print("\nInstance of D:")
who(d)
```

    4542787992 <class '__main__.D'>
    C.method
    4542787992 <class '__main__.D'>
    C.method
    4542787992 <class '__main__.D'>
    B.method
    4542787992 <class '__main__.D'>
    A.method
    4542788048 <class '__main__.C'>
    A.method
    
    Instance of D:
    4542787992 <class '__main__.D'>


当然我们也可以在外部使用 `super()` 方法，只是不能再用缺省参数的形式，因为在外部的命名空间中不再存在 `__class__` 和 `self`：


```python
super(D, d).method() # calling D's parent's method with instance d
```

    4542787992 <class '__main__.D'>
    C.method


上面的例子可以用下图来描述：

```
+----------+
| A        |
+----------+
| method() <---------------+ super(B,self)
+----------+               |
                           |
+----------+               +----------+
| B        |               | D        |
+----------+ super(C,self) +----------+
| method() <---------------+ method() |
+----------+               +----------+
                           |
+----------+               |
| C        |               |
+----------+               | super(D,self)
| method() <---------------+
+----------+
```

可以认为 `super()` 方法通过向父类方向回溯给我们找到了变量搜寻的起点，但是这个回溯的顺序是如何确定的呢？上面的例子中继承关系是 `object->A->B->C->D` 的顺序，如果是比较复杂的继承关系呢？


```python
class A(object):
    pass
class B(A):
    def method(self):
        print("B's method")
class C(A):
    def method(self):
        print("C's method")
class D(B, C):
    def __init__(self):
        super().method()
class E(C, B):
    def __init__(self):
        super().method()
        
d = D()
e = E()
```

    B's method
    C's method


Python 中提供了一个类方法 `mro()` 可以指定搜寻的顺序，`mro` 是**Method Resolution Order** 的缩写，它是类方法而不是实例方法，可以通过重载 `mro()` 方法改变继承中的方法解析顺序，但这需要在元类中完成，在这里只看一下其结果：


```python
D.mro()
```




    [__main__.D, __main__.B, __main__.C, __main__.A, object]




```python
E.mro()
```




    [__main__.E, __main__.C, __main__.B, __main__.A, object]



`super()` 方法就是沿着 `mro()` 给出的顺序向上寻找起点的：


```python
super(D, d).method()
super(E, e).method()
```

    B's method
    C's method



```python
super(C, e).method()
super(B, d).method()
```

    B's method
    C's method


### 总结

`super()` 方法解决了**类->实例**实践过程中关于命名空间的一些问题，而关于生成对象的流程，我们知道初始化实例是通过类的 `__init__()` 方法完成的，在此之前可能涉及到一些其它的准备工作，包括上面提到的 `mro()` 方法以及关键的**元类->类**的过程，将在后面一篇中继续介绍。
