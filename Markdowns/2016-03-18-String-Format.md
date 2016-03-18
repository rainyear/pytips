
## Python 字符串的格式化

相信很多人在格式化字符串的时候都用`"%s" % v`的语法，[PEP 3101](https://www.python.org/dev/peps/pep-3101/) 提出一种更先进的格式化方法 `str.format()` 并成为 Python 3 的标准用来替换旧的 `%s` 格式化语法，CPython 从 2.6 开始已经实现了这一方法（其它解释器未考证）。

### `format()`

新的 `format()` 方法其实更像是一个简略版的模板引起（Template Engine），功能非常丰富，官方文档对其语法的描述如下：


```python
"""
replacement_field ::=  "{" [field_name] ["!" conversion] [":" format_spec] "}"
field_name        ::=  arg_name ("." attribute_name | "[" element_index "]")*
arg_name          ::=  [identifier | integer]
attribute_name    ::=  identifier
element_index     ::=  integer | index_string
index_string      ::=  <any source character except "]"> +
conversion        ::=  "r" | "s" | "a"
format_spec       ::=  <described in the next section>
"""
pass # Donot output
```

我将其准换成[铁路图](https://en.wikipedia.org/wiki/Syntax_diagram)的形式，（可能）更直观一些：

![replacement_field.jpg](http://7xiijd.com1.z0.glb.clouddn.com/replacement_field.jpg)

模板中替换变量用 `{}` 包围，且由 `:` 分为两部分，其中后半部分 `format_spec` 在后面会单独讨论。前半部分有三种用法：

1. 空
2. 代表位置的数字
3. 代表keyword的标识符

这与函数调用的参数类别是一致的：


```python
print("{} {}".format("Hello", "World"))
# is equal to...
print("{0} {1}".format("Hello", "World"))
print("{hello} {world}".format(hello="Hello", world="World"))

print("{0}{1}{0}".format("H", "e"))
```

    Hello World
    Hello World
    Hello World
    HeH


除此之外，就像在[0x05 函数参数与解包](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-11-Arguments-and-Unpacking.ipynb)中提到的一样，`format()` 中也可以直接使用解包操作：


```python
print("{lang}.{suffix}".format(**{"lang": "Python", "suffix": "py"}))
print("{} {}".format(*["Python", "Rocks"]))
```

    Python.py
    Python Rocks


在模板中还可以通过 `.identifier` 和 `[key]` 的方式获取变量内的属性或值（需要注意的是 `"{}{}"` 相当于 `"{0}{1}"`）：


```python
data = {'name': 'Python', 'score': 100}
print("Name: {0[name]}, Score: {0[score]}".format(data)) # 不需要引号

langs = ["Python", "Ruby"]
print("{0[0]} vs {0[1]}".format(langs))

print("\n====\nHelp(format):\n {.__doc__}".format(str.format))
```

    Name: Python, Score: 100
    Python vs Ruby
    
    ====
    Help(format):
     S.format(*args, **kwargs) -> str
    
    Return a formatted version of S, using substitutions from args and kwargs.
    The substitutions are identified by braces ('{' and '}').


### 强制转换

可以通过 `!` + `r|s|a` 的方式对替换的变量进行强制转换：

1. `"{!r}"` 对变量调用 `repr()`
2. `"{!s}"` 对变量调用 `str()`
3. `"{!a}"` 对变量调用 `ascii()`

### 格式

最后 `:` 之后的部分定义输出的样式：

![format_spec.jpg](http://7xiijd.com1.z0.glb.clouddn.com/format_spec.jpg)

`align` 代表对齐方向，通常要配合 `width` 使用，而 `fill` 则是填充的字符（默认为空白）：


```python
for align, text in zip("<^>", ["left", "center", "right"]):
    print("{:{fill}{align}16}".format(text, fill=align, align=align))
    
print("{:0=10}".format(100)) # = 只允许数字
```

    left<<<<<<<<<<<<
    ^^^^^center^^^^^
    >>>>>>>>>>>right
    0000000100


同时可以看出，样式设置里面可以嵌套 `{}` ，但是必须通过 keyword 指定，且只能嵌套一层。

接下来是符号样式：`+|-|' '` 分别指定数字是否需要强制符号（其中空格是指在正数的时候不显示 `+` 但保留一位空格）：


```python
print("{0:+}\n{1:-}\n{0: }".format(3.14, -3.14))
```

    +3.14
    -3.14
     3.14


`#` 用于表示特殊格式的数字（二进制、十六进制等）是否需要前缀符号；`,` 也是用于表示数字时是否需要在千位处进行分隔；`0` 相当于前面的 `{:0=}` 右对齐并用 `0` 补充空位：


```python
print("Binary: {0:b} => {0:#b}".format(3))

print("Large Number: {0:} => {0:,}".format(1.25e6))

print("Padding: {0:16} => {0:016}".format(3))
```

    Binary: 11 => 0b11
    Large Number: 1250000.0 => 1,250,000.0
    Padding:                3 => 0000000000000003


最后两个就是我们熟悉的小数点精度 `.n` 和格式化类型了，这里仅给出一些示例，详细内容可以查阅[文档](https://docs.python.org/3/library/string.html#formatexamples)：


```python
from math import pi
print("pi = {pi:.2}, also = {pi:.7}".format(pi=pi))
```

    pi = 3.1, also = 3.141593


**Integer**


```python
for t in "b c d #o #x #X n".split():
    print("Type {0:>2} of {1} shows: {1:{t}}".format(t, 97, t=t))
```

    Type  b of 97 shows: 1100001
    Type  c of 97 shows: a
    Type  d of 97 shows: 97
    Type #o of 97 shows: 0o141
    Type #x of 97 shows: 0x61
    Type #X of 97 shows: 0X61
    Type  n of 97 shows: 97


**Float**


```python
for t, n in zip("eEfFgGn%", [12345, 12345, 1.3, 1.3, 1, 2, 3.14, 0.985]):
    print("Type {} shows: {:.2{t}}".format(t, n, t=t))
```

    Type e shows: 1.23e+04
    Type E shows: 1.23E+04
    Type f shows: 1.30
    Type F shows: 1.30
    Type g shows: 1
    Type G shows: 2
    Type n shows: 3.1
    Type % shows: 98.50%


**String (default)**


```python
try:
    print("{:s}".format(123))
except:
    print("{}".format(456))
```

    456

