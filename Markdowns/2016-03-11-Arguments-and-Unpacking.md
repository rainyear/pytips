
### 函数调用的参数规则与解包

Python 的函数在声明参数时大概有下面 4 种形式：

1. 不带默认值的：`def func(a): pass`
2. 带有默认值的：`def func(a, b = 1): pass`
3. 任意位置参数：`def func(a, b = 1, *c): pass`
4. 任意键值参数：`def func(a, b = 1, *c, **d): pass`

在调用函数时，有两种情况：

1. 没有关键词的参数：`func("G", 20)`
2. 带有关键词的参数：`func(a = "G", b = 20)`（其中带有关键词调用可以不考虑顺序：`func(b = 20, a = "G"`）

当然，这两种情况是可以混用的：`func("G", b = 20)`，但最重要的一条规则是**位置参数不能在关键词参数之后出现**：


```python
def func(a, b = 1):
    pass
func(a = "G", 20) # SyntaxError 语法错误
```


      File "<ipython-input-1-3ca775953480>", line 3
        func(a = "G", 20) # SyntaxError 语法错误
                     ^
    SyntaxError: positional argument follows keyword argument



另外一条规则是：**位置参数优先权**：


```python
def func(a, b = 1):
    pass
func(20, a = "G") # TypeError 对参数 a 重复赋值
```


    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-2-cbaa9a1fe24d> in <module>()
          1 def func(a, b = 1):
          2     pass
    ----> 3 func(20, a = "G") # TypeError 对参数 a 重复赋值
    

    TypeError: func() got multiple values for argument 'a'


最保险的方法就是全部采用关键词参数。

#### 任意参数

任意参数可以接受任意数量的参数，其中`*a`的形式代表任意数量的位置参数，`**d`代表任意数量的关键词参数：


```python
def concat(*lst, sep = "/"):
    return sep.join((str(i) for i in lst))

print(concat("G", 20, "@", "Hz", sep = ""))
```

    G20@Hz


上面的这个`def concat(*lst, sep = "/")`的语法是[PEP 3102](https://www.python.org/dev/peps/pep-3102/)提出的，在 Python 3.0 之后实现。这里的关键词函数必须明确指明，不能通过位置推断：


```python
print(concat("G", 20, "-")) # Not G-20
```

    G/20/-


`**d`则代表任意数量的关键词参数


```python
def dconcat(sep = ":", **dic):
    for k in dic.keys():
        print("{}{}{}".format(k, sep, dic[k]))

dconcat(hello = "world", python = "rocks", sep = "~")
```

    hello~world
    python~rocks


#### Unpacking

Python 3.5 添加的新特性（[PEP 448](https://www.python.org/dev/peps/pep-0448/)），使得`*a`、`**d`可以在函数参数之外使用：


```python
print(*range(5))
lst = [0, 1, 2, 3]
print(*lst)

a = *range(3), # 这里的逗号不能漏掉
print(a)

d = {"hello": "world", "python": "rocks"}
print({**d}["python"])
```

    0 1 2 3 4
    0 1 2 3
    (0, 1, 2)
    rocks


所谓的解包（Unpacking）实际上可以看做是去掉`()`的元组或者是去掉`{}`的字典。这一语法也提供了一个更加 Pythonic 地合并字典的方法：


```python
user = {'name': "Trey", 'website': "http://treyhunner.com"}
defaults = {'name': "Anonymous User", 'page_name': "Profile Page"}

print({**defaults, **user})
```

    {'page_name': 'Profile Page', 'name': 'Trey', 'website': 'http://treyhunner.com'}


在函数调用的时候使用这种解包的方法则是 Python 2.7 也可以使用的：


```python
print(concat(*"ILovePython"))
```

    I/L/o/v/e/P/y/t/h/o/n


#### 参考

1. [The Idiomatic Way to Merge Dictionaries in Python](https://treyhunner.com/2016/02/how-to-merge-dictionaries-in-python/)
