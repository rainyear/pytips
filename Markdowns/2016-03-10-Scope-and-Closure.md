
### 闭包（Closure）

> 在计算机科学中，闭包（英语：Closure），又称词法闭包（Lexical Closure）或函数闭包（function closures），是引用了自由变量的函数。这个被引用的自由变量将和这个函数一同存在，即使已经离开了创造它的环境也不例外。
[[维基百科::闭包（计算机科学）](https://zh.wikipedia.org/wiki/闭包_%28计算机科学%29)]

[0x02 Python 中的函数式编程](https://github.com/rainyear/pytips/blob/master/Markdowns/2016-03-08-Functional-Programming-in-Python.md) 本来也应该包括闭包的概念，但是我觉得闭包更重要的是对**作用域（Scope）**的理解，因此把它单独列出来，同时可以理顺一下 Python 的作用域规则。

闭包的概念最早出现在函数式编程语言中，后来被一些命令式编程语言所借鉴。尤其是在一些函数作为一等公民的语言中，例如JavaScript就经常用到（在JavaScript中函数几乎可以当做“特等公民”看待），我之前也写过一篇关于JavaScript闭包的文章（[图解Javascript上下文与作用域](http://blog.rainy.im/2015/07/04/scope-chain-and-prototype-chain-in-js/)），实际上闭包并不是太复杂的概念，但是可以借助闭包更好地理解不同语言的作用域规则。

#### 命名空间与作用域

[0x00 The Zen of Python](https://github.com/rainyear/pytips/blob/master/Markdowns/2016-03-06-The-Zen-of-Python.md)的最后一句重点强调命名空间的概念，我们可以把命名空间看做一个大型的字典类型（Dict），里面包含了所有变量的名字和值的映射关系。在 Python 中，作用域实际上可以看做是“**在当前上下文的位置，获取命名空间变量的规则**”。在 Python 代码执行的任意位置，都至少存在三层嵌套的作用域：

1. 最内层作用域，最早搜索，包含所有局部变量**（Python 默认所有变量声明均为局部变量）**
2. 所有包含当前上下文的外层函数的作用域，由内而外依次搜索，这里包含的是**非局部**也**非全局**的变量
3. 一直向上搜索，直到当前模块的全局变量
4. 最外层，最后搜索的，内置（built-in）变量

在任意执行位置，可以将作用域看成是对下面这样一个命名空间的搜索：


```python
scopes = {
    "local": {"locals": None,
             "non-local": {"locals": None,
                          "global": {"locals": None,
                                    "built-in": ["built-ins"]}}},
}
```

除了默认的局部变量声明方式，Python 还有`global`和`nonlocal`两种类型的声明（**`nonlocal`是Python 3.x之后才有，2.7没有**），其中 `global` 指定的变量直接**指向**(3)当前模块的全局变量，而`nonlocal`则指向(2)最内层之外，`global`以内的变量。这里需要强调指向（references and assignments）的原因是，普通的局部变量对最内层局部作用域之外只有**只读(read-only)**的访问权限，比如下面的例子：


```python
x = 100
def main():
    x += 1
    print(x)
main()
```


    ---------------------------------------------------------------------------

    UnboundLocalError                         Traceback (most recent call last)

    <ipython-input-2-9ed43e483a17> in <module>()
          3     x += 1
          4     print(x)
    ----> 5 main()
    

    <ipython-input-2-9ed43e483a17> in main()
          1 x = 100
          2 def main():
    ----> 3     x += 1
          4     print(x)
          5 main()


    UnboundLocalError: local variable 'x' referenced before assignment


这里抛出`UnboundLocalError`，是因为`main()`函数内部的作用域对于全局变量`x`仅有只读权限，想要在`main()`中对`x`进行改变，不会影响全局变量，而是会创建一个新的局部变量，显然无法对还未创建的局部变量直接使用`x += 1`。如果想要获得全局变量的完全引用，则需要`global`声明：


```python
x = 100
def main():
    global x
    x += 1
    print(x)
    
main()
print(x) # 全局变量已被改变
```

    101
    101


#### Python 闭包

到这里基本上已经了解了 Python 作用域的规则，那么我们来仿照 JavaScript 写一个计数器的闭包：


```python
"""
/* JavaScript Closure example */
var inc = function(){  
  var x = 0;
  return function(){
    console.log(x++);
  };
};
var inc1 = inc()
var inc2 = inc()
"""

# Python 3.5
def inc():
    x = 0
    def inner():
        nonlocal x
        x += 1
        print(x)
    return inner
inc1 = inc()
inc2 = inc()

inc1()
inc1()
inc1()
inc2()
```

    1
    2
    3
    1


对于还没有`nonlocal`关键字的 Python 2.7，可以通过一点小技巧来规避局部作用域只读的限制：


```python
# Python 2.7
def inc():
    x = [0]
    def inner():
        x[0] += 1
        print(x[0])
    return inner
inc1 = inc()
inc2 = inc()

inc1()
inc1()
inc1()
inc2()
```

    1
    2
    3
    1


上面的例子中，`inc1()`是在全局环境下执行的，虽然全局环境是不能向下获取到`inc()`中的局部变量`x`的，但是我们返回了一个`inc()`内部的函数`inner()`，而`inner()`对`inc()`中的局部变量是有访问权限的。也就是说`inner()`将`inc()`内的局部作用域打包送给了`inc1`和`inc2`，从而使它们各自独立拥有了一块封闭起来的作用域，不受全局变量或者任何其它运行环境的影响，因此称为闭包。

闭包函数都有一个`__closure__`属性，其中包含了它所引用的上层作用域中的变量：


```python
print(inc1.__closure__[0].cell_contents)
print(inc2.__closure__[0].cell_contents)
```

    [3]
    [1]


#### 参考

1. [9.2. Python Scopes and Namespaces](https://docs.python.org/3/tutorial/classes.html#python-scopes-and-namespaces)
2. [Visualize Python Execution](http://www.pythontutor.com/visualize.html#mode=edit)
3. [Wikipedia::Closure](https://en.wikipedia.org/wiki/Closure_%28computer_programming%29)
