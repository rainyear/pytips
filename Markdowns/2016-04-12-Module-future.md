
### Python `__future__` 模块

我们经常从一些组织良好的 Python 项目中看到 `__future__` 的身影，例如：


```python
from math import sin
from __future__ import absolute_import, division, print_function, with_statement
```

`__future__` 模块由 [PEP 236](https://www.python.org/dev/peps/pep-0236/) 提出并加入到 Python 2.1，其存在的主要原因是 Python 的版本升级经常会增加一些新的特性，而 `__future__` 模块将一些新版本中将会增加的新的特性进行声明，同时使得旧版本可以使用这些新的语法特性。当然，如果你用的是 Python 2.1 以前的版本，是没办法使用 `__future__` 的。`__future__` 模块的导入一定要放在最上方，也就是在所有其它模块之前导入。

根据 Python 3.5 中 `__future__` 的[源文件](https://hg.python.org/cpython/file/3.5/Lib/__future__.py)，一共包括下面几个新特性：

![__future__](http://qncdn.rainy.im/__future__.png)

表中第一列包含了所有可以从 `__future__` 中导入的特性，`optional in` 中的版本号为**最低可使用的版本**，`mandatory in` 中的版本号为**已经实现，无需从 `__future__` 导入** 的版本号。最后一列是每个新特性所对应的 PEP 及简单描述。下面主要看一下几个 3.0 之后添加的属性。

**division**

在 3.0 之前，除号 `/` 的用法都是整除，例如在 2.7 中，`8/7=1`，如果需要浮点数相除，需要显式使用浮点数类型：`8./7=1.1428571428571428`，如果是使用变量相除，则需要 `a*1.0/7`。而在 3.0 以后的版本，整除和浮点数相除分别使用 `//` 和 `/` 符号：


```python
from __future__ import division
print(8//7)
print(8/7)
```

    1
    1.1428571428571428


**absolute_import**

[PEP 328](https://www.python.org/dev/peps/pep-0328/) 主要添加了多行导入和绝对/相对导入的特性，可以使用括号而不是 `\` 来进行多行导入。相对导入则可以使用 `.` 为标记导入相对目录中的模块，具体可以参考这篇文章：[Python导入模块的几种姿势](http://codingpy.com/article/python-import-101/)。


```python
from __future__ import absolute_import

from tkinter import (Tk, Frame, Button, Entry, Canvas, Text,
    LEFT, DISABLED, NORMAL, RIDGE, END)
```

**print_function**

我们都知道 3.0 以后 `print` 变为函数而不再是声明语句，而是一个内置函数，并且可以接受除打印内容以外的参数：

`print(*args, sep=' ', end='\n', file=None)`


```python
from __future__ import print_function
print("Hello", "World", sep=',', end='!')
```

    Hello,World!

**generator_stop**

在生成器内部使用 `raise StopIteration` 意味着生成器终止，但这样写会造成一些隐藏的 BUG（见 [PEP 479](https://www.python.org/dev/peps/pep-0479/)），新的特性要求杜绝这一用法并抛出 `RuntimeError`，若要终止生成器，请使用 `return`。


```python
from __future__ import generator_stop
def gen(n):
    i = 0
    while i < n:
        yield i
        i += 1
        if i % 2 == 0:
            raise StopIteration
            # use return
        
g = gen(3)
print(next(g))
print(next(g))
try:
    print(next(g))
except RuntimeError as err:
    print(err)
```

    0
    1
    generator raised StopIteration


### 彩蛋

为了吸引更多 Java、C 等语言的用户，使他们更快适应 Python 代码缩进的语法同时又能兼容他们不可抑制地输入 `{}` 的冲动，`__future__` 还提供了 `braces` 特性，使花括号可以被正确解析！

```py
from __future__ import braces
```
