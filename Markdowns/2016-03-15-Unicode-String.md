
### Python 字符串

所有用过 Python (2&3)的人应该都看过下面两行错误信息：

> `UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)`

> `UnicodeDecodeError: 'utf-8' codec can't decode bytes in position 0-1: invalid continuation byte`

这就是 Python 界的"锟斤拷"！

今天和接下来几期的内容将主要关注 Python 中的字符串（`str`）、字节（`bytes`）及两者之间的相互转换（`encode`/`decode`）。也许不能让你突然间解决所有乱码问题，但希望可以帮助你迅速找到问题所在。

### 定义

Python 中对字符串的定义如下：

> Textual data in Python is handled with `str` objects, or strings. Strings are immutable sequences of Unicode code points.

Python 3.5 中字符串是由一系列 Unicode 码位（code point）所组成的**不可变序列**：


```python
('S' 'T' 'R' 'I' 'N' 'G')
```




    'STRING'



**不可变**是指无法对字符串本身进行更改操作：


```python
s = 'Hello'
print(s[3])
s[3] = 'o'
```

    l



    ---------------------------------------------------------------------------

    TypeError                                 Traceback (most recent call last)

    <ipython-input-2-ce8cf24852f9> in <module>()
          1 s = 'Hello'
          2 print(s[3])
    ----> 3 s[3] = 'o'
    

    TypeError: 'str' object does not support item assignment


而**序列（sequence）**则是指字符串继承序列类型（`list/tuple/range`）的通用操作：


```python
[i.upper() for i in "hello"]
```




    ['H', 'E', 'L', 'L', 'O']



至于 Unicode 暂时可以看作一张非常大的地图，这张地图里面记录了世界上所有的符号，而码位则是每个符号所对应的坐标（具体内容将在后面的几期介绍）。


```python
s = '雨'
print(s)
print(len(s))
print(s.encode())
```

    雨
    1
    b'\xe9\x9b\xa8'


### 常用操作

- **`len`**：字符串长度；
- **`split` & `join`**
- **`find` & `index`**
- **`strip`**
- **`upper` & `lower` & `swapcase` & `title` & `capitalize`**
- **`endswith` & `startswith` & `is*`**
- **`zfill`**


```python
# split & join
s = "Hello world!"
print(",".join(s.split())) # 常用的切分 & 重组操作

"https://github.com/rainyear/pytips".split("/", 2) # 限定切分次数
```

    Hello,world!





    ['https:', '', 'github.com/rainyear/pytips']




```python
s = "coffee"
print(s.find('f'))    # 从左至右搜索，返回第一个下标
print(s.rfind('f'))   # 从右至左搜索，返回第一个下表

print(s.find('a'))    # 若不存在则返回 -1
print(s.index('a'))   # 若不存在则抛出 ValueError，其余与 find 相同
```

    2
    3
    -1



    ---------------------------------------------------------------------------

    ValueError                                Traceback (most recent call last)

    <ipython-input-6-59556fd9319f> in <module>()
          4 
          5 print(s.find('a'))    # 若不存在则返回 -1
    ----> 6 print(s.index('a'))   # 若不存在则抛出 ValueError，其余与 find 相同
    

    ValueError: substring not found



```python
print(" hello world    ".strip())
print("helloworld".strip("heo"))
print("["+"          i         ".lstrip() +"]")
print("["+"          i         ".rstrip() +"]")
```

    hello world
    lloworld
    [i         ]
    [          i]



```python
print("{}\n{}\n{}\n{}\n{}".format(
    "hello, WORLD".upper(),
    "hello, WORLD".lower(),
    "hello, WORLD".swapcase(),
    "hello, WORLD".capitalize(),
    "hello, WORLD".title()))
```

    HELLO, WORLD
    hello, world
    HELLO, world
    Hello, world
    Hello, World



```python
print("""
{}|{}
{}|{}
{}|{}
{}|{}
{}|{}
{}|{}
""".format(
    "Python".startswith("P"),"Python".startswith("y"),
    "Python".endswith("n"),"Python".endswith("o"),
    "i23o6".isalnum(),"1 2 3 0 6".isalnum(),
    "isalpha".isalpha(),"isa1pha".isalpha(),
    "python".islower(),"Python".islower(),
    "PYTHON".isupper(),"Python".isupper(),
))
```

    
    True|False
    True|False
    True|False
    True|False
    True|False
    True|False
    



```python
"101".zfill(8)
```




    '00000101'



**`format` / `encode`**

格式化输出 `format` 是非常有用的工具，将会单独进行介绍；`encode` 会在 `bytes-decode-Unicode-encode-bytes` 中详细介绍。
