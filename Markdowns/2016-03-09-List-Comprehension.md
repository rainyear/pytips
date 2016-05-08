
### 0x03 - Python 列表推导

**[0x02](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-08-Functional-Programming-in-Python.ipynb)** 中提到的 `map`/`filter` 方法可以通过简化的语法快速构建我们需要的列表（或其它可迭代对象），与它们功能相似的，Python 还提供**列表推导（List Comprehension）**的语法。最初学 Python 的时候，我只是把这种语法当做一种**语法糖**，可以用来快速构建特定的列表，后来学习 Haskell 的时候才知道这种形式叫做 List Comprehension（中文我好像没有找到固定的翻译，有翻译成**列表速构、列表解析**之类的，但意思上都是在定义列表结构的时候按照一定的规则进行推导，而不是穷举所有元素）。

这种列表推导与数学里面集合的表达形式有些相似，例如$[0, 10)$之间偶数集合可以表示为：

$$\left\{x\ |\ x \in N, x \lt 10, x\ mod\ 2\ ==\ 0\right\}$$

翻译成 Python 表达式为：


```python
evens = [x for x in range(10) if x % 2 == 0]
print(evens)
```

    [0, 2, 4, 6, 8]


这与`filter`效果一样：


```python
fevens = filter(lambda x: x % 2 == 0, range(10))
print(list(fevens) == evens)
```

    True


同样，列表推导也可以实现`map`的功能：


```python
squares = [x ** 2 for x in range(1, 6)]
print(squares)

msquares = map(lambda x: x ** 2, range(1, 6))
print(list(msquares) == squares)
```

    [1, 4, 9, 16, 25]
    True


相比之下，列表推导的语法更加直观，因此更 Pythonic 的写法是在可以用列表推导的时候尽量避免`map`/`filter`。

除了上面简单的迭代、过滤推导之外，列表推导还支持嵌套结构：


```python
cords = [(x, y) for x in range(3) for y in range(3) if x > 0]
print(cords)

# 相当于
lcords = []
for x in range(3):
    for y in range(3):
        if x > 0:
            lcords.append((x, y))
            
print(lcords == cords)
```

    [(1, 0), (1, 1), (1, 2), (2, 0), (2, 1), (2, 2)]
    True


#### 字典与集合的推导

这样一比较更加能够突出列表推导的优势，但是当嵌套的循环超过2层之后，列表推导语法的可读性也会大大下降，所以当循环嵌套层数增加时，还是建议用直接的语法。

Python 中除了列表（List）可以进行列表推导之外，字典（Dict）、集合（Set）同样可以：


```python
dns = {domain : ip
       for domain in ["github.com", "git.io"]
       for ip in ["23.22.145.36", "23.22.145.48"]}
print(dns)

names = {name for name in ["ana", "bob", "catty", "octocat"] if len(name) > 3}
print(names)
```

    {'github.com': '23.22.145.48', 'git.io': '23.22.145.48'}
    {'octocat', 'catty'}


#### 生成器

**[0x01](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-07-iterator-and-generator.ipynb)**中提到的生成器（Generator），除了在函数中使用 `yield` 关键字之外还有另外一种隐藏方法，那就是对元组（Tuple）使用列表推导：


```python
squares = (x for x in range(10) if x % 2 == 0)
print(squares)

print(next(squares))
next(squares)

for i in squares:
    print(i)
```

    <generator object <genexpr> at 0x1104fbba0>
    0
    4
    6
    8

