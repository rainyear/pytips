
### Python 描述符

> 本篇主要关于三个常用内置方法：`property()`，`staticmethod()`，`classmethod()`

在 Python 语言的设计中，通常的语法操作最终都会转化为方法调用，例如：


```python
a = 1
b = 2
print("a + b = {}".format(a+b))

# 相当于
print("a.__add__(b) = {}".format(a.__add__(b)))
```

    a + b = 3
    a.__add__(b) = 3


Python 中的**描述符（Descriptor）**就是将对象属性的获取、赋值以及删除等行为转换为方法调用的协议：

```py
descr.__get__(self, obj, type=None) --> value

descr.__set__(self, obj, value) --> None

descr.__delete__(self, obj) --> None
```

例如我们要获取一个对象的属性，可以通过`o.x`的方式取得：


```python
class Int:
    ctype = "Class::Int"
    def __init__(self, val):
        self._val = val
        
a = Int(1)
print(a.ctype)
```

    Class::Int


而通过`.`的方式寻找属性的值实际上调用了`object.__getattribute__(self, name)`方法：


```python
class Int:
    ctype = "Class::Int"
    def __init__(self, val):
        self._val = val
    def __getattribute__(self, name):
        print("👿 doesn't want to give `{}' to you!".format(name))
        return "🐍"
a = Int(2)
print(a.ctype)
```

    👿 doesn't want to give `ctype' to you!
    🐍


而这里的`__getattribute__(self, name)`方法实际上就是将`.`的属性获取方法转化为描述符协议定义的`descr.__get__(self, key)`：


```python
class Str:
    def __init__(self, val):
        self._val = val
    def __get__(self, name, ctype=None):
        print("You can __get__ anything from here!")
        return self._val
class Int:
    ctype = Str("Class::Int")
    def __init__(self, val):
        self._val = val
    def __getattribute__(self, name):
        return type(self).__dict__[name].__get__(None, type(self))
a = Int(2)
print(a.ctype)
```

    You can __get__ anything from here!
    Class::Int


这里的 `a.ctype = (Int.__dict__['ctype']).__get__(None, Int)`，即通过描述符的方式获取了 `ctype` 属性的值。同样的道理，你也可以通过 `descr.__set__(self, obj, val)` 设置属性的值：


```python
class Str:
    def __init__(self, val):
        self._val = val
    def __get__(self, name, ctype=None):
        print("You can __get__ anything from here!")
        return self._val
    def __set__(self, name, val):
        print("You can __set__ anything to me!")
        self._val = val
class Int:
    ctype = Str("Class::Int")
    def __init__(self, val):
        self._val = val
a = Int(3)
print(a.ctype)
a.ctype = "Class::Float"
print(a.ctype)
```

    You can __get__ anything from here!
    Class::Int
    You can __set__ anything to me!
    You can __get__ anything from here!
    Class::Float


将这些取值、赋值的操作转换为方法调用让我们有办法在做这些操作的过程中插入一些小动作，这么好用的东西自然是已加入豪华内置函数阵容，正是我们常见的

- `property()`
- `classmethod()`
- `staticmethod()`



**property**

`property(fget=None, fset=None, fdel=None, doc=None)` 方法简化了上面的操作：


```python
class Int:
    def __init__(self, val):
        self._val = val
        self._ctype = None
        
    def get_ctype(self):
        print("INFO: You can get `ctype`")
        return self._ctype
    def set_ctype(self, val):
        print("INFO: You're setting `ctype` =", val)
        self._ctype=val
    ctype = property(fget=get_ctype, fset=set_ctype, doc="Property `ctype`")
    
a = Int(4)
print(a.ctype)
a.ctype = "Class::Int"
print(a.ctype)
```

    INFO: You can get `ctype`
    None
    INFO: You're setting `ctype` = Class::Int
    INFO: You can get `ctype`
    Class::Int


显然，更方便一些的用法是将 `property` 当做修饰器：


```python
class Int:
    _ctype = None
    def __init__(self, val):
        self._val = val
    @property
    def ctype(self):
        print("INFO: You can get `ctype` from me!")
        return self._ctype
    @ctype.setter
    def ctype(self, val):
        print("INFO: You're setting `ctype` =", val)
        self._ctype = val
a = Int(5)
print(a.ctype)
a.ctype = "Class::Int"
print(a.ctype)
```

    INFO: You can get `ctype` from me!
    None
    INFO: You're setting `ctype` = Class::Int
    INFO: You can get `ctype` from me!
    Class::Int


**staticmethod & classmethod**

顾名思义，`property` 是关于属性的全部操作，如果是要获取类中的方法，则需要用到 `staticmethod` 和 `classmethod`。顾名思义，`staticmethod` 将方法变成静态方法，即类和实例都可以访问，如果不用 `staticmethod` 我们可以用下面这种别扭的方法实现：


```python
class Int:
    def __init__(self, val):
        self._val = val
    def _get_ctype(self=None):
        print("INFO: You can get `ctype` from here!")
        return "Class::Int"
    
    @staticmethod
    def get_ctype():
        print("INFO: You can get `ctype` from here!")
        return "Class::StaticInt"       
    
a = Int(6)
print(a._get_ctype())
print(Int._get_ctype())

print(a.get_ctype())
print(Int.get_ctype())
```

    INFO: You can get `ctype` from here!
    Class::Int
    INFO: You can get `ctype` from here!
    Class::Int
    INFO: You can get `ctype` from here!
    Class::StaticInt
    INFO: You can get `ctype` from here!
    Class::StaticInt


可以看到，静态方法与类和实例无关，也就不再（不能）需要 `self` 关键词；与之相反，当我们需要在方法中保留类（而非实例）的引用时，则需要用 `classmethod`：


```python
class Int:
    _ctype = ""
    def __init__(self, val):
        self._val = val
        
    @classmethod
    def set_ctype(klass, t):
        klass._ctype = t
        return "{}.ctype = {}".format(klass.__name__, t)
a = Int(7)
print(a.set_ctype("Class::Int"))
print(Int.set_ctype("Class::Float"))
b = Int(8)
print(b._ctype)
```

    Int.ctype = Class::Int
    Int.ctype = Class::Float
    Class::Float


### 总结

Python 的描述符给出一种通过方法调用来实现属性（方法）获取、赋值等操作的规则，通过这一规则可以方便我们深入程序内部并实施操控，因此 `property/staticmethod/classmethod` 在 Python 是通过底层（如 CPython 中的 C）实现的，如果想要进一步深入了解其实现原理，可以访问参考链接的教程，其中包括了这三个内置方法的 Python 实现版本，我也把它们 copy 过来方便查看。

### 参考

1. [Descriptor HowTo Guide](https://docs.python.org/3/howto/descriptor.html?highlight=descriptors#static-methods-and-class-methods)


```python
class Property(object):
    "Emulate PyProperty_Type() in Objects/descrobject.c"

    def __init__(self, fget=None, fset=None, fdel=None, doc=None):
        self.fget = fget
        self.fset = fset
        self.fdel = fdel
        if doc is None and fget is not None:
            doc = fget.__doc__
        self.__doc__ = doc

    def __get__(self, obj, objtype=None):
        if obj is None:
            return self
        if self.fget is None:
            raise AttributeError("unreadable attribute")
        return self.fget(obj)

    def __set__(self, obj, value):
        if self.fset is None:
            raise AttributeError("can't set attribute")
        self.fset(obj, value)

    def __delete__(self, obj):
        if self.fdel is None:
            raise AttributeError("can't delete attribute")
        self.fdel(obj)

    def getter(self, fget):
        return type(self)(fget, self.fset, self.fdel, self.__doc__)

    def setter(self, fset):
        return type(self)(self.fget, fset, self.fdel, self.__doc__)

    def deleter(self, fdel):
        return type(self)(self.fget, self.fset, fdel, self.__doc__)

class StaticMethod(object):
    "Emulate PyStaticMethod_Type() in Objects/funcobject.c"

    def __init__(self, f):
        self.f = f
        
    def __get__(self, obj, objtype=None):
        return self.f

class ClassMethod(object):
    "Emulate PyClassMethod_Type() in Objects/funcobject.c"
    
    def __init__(self, f):
        self.f = f

    def __get__(self, obj, klass=None):
        if klass is None:
            klass = type(obj)
        def newfunc(*args):
            return self.f(klass, *args)
        return newfunc
```
