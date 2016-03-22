
## Python 知之深浅

Python 中的对象分为两种：可变对象（mutable）和不可变对象（immutable）。不可变对象包括int，float，long，str，tuple等，可变对象包括list，set，dict等。在 Python 中，赋值（assignment, `=`）的过程仅仅是：

1. 创建一个（某个值的）对象；
2. 将变量名指向（引用）这个对象。

这就像 C 语言中指针的概念，只不过更灵活地是 Python 中的变量随时可以指向其它对象（不分类型），其它变量也可以指向这一对象。如果这一对象是可变的，那么对其中一个引用变量的改变会影响其它变量：


```python
lst = [1, 2, 3]
s = lst
s.pop()
print(lst)

d = {'a': 0}
e = d
e['b'] = 1
print(d)
```

    [1, 2]
    {'b': 1, 'a': 0}


如果你不是刻意想要这样做（实际也很少会要这样操作），那么就可能导致一些意想不到的错误（尤其是在传递参数给函数的时候）。为了解决这一麻烦，最简单的方法就是不直接变量指向现有的对象，而是生成一份新的 copy 赋值给新的变量，有很多种语法可以实现：


```python
lst = [1,2,3]

llst = [lst,
        lst[:],
        lst.copy(),
        [*lst]] # invalid in 2.7
for i, v in enumerate(llst):
    v.append("#{}".format(i))
print(lst)

d = {"a": 0}
dd = [d,
      d.copy(),
      {**d}] # invalid in 2.7
for i, v in enumerate(dd):
    v['dd'] = "#{}".format(i)
print(d)
```

    [1, 2, 3, '#0']
    {'dd': '#0', 'a': 0}


### `deep` vs `shallow`

上面给出的这些 copy 的例子比较简单，都没有嵌套的情况出现，如果这里的可变对象中还包含其它可变对象，结果会怎样呢：


```python
lst = [0, 1, [2, 3]]

llst = [lst,
        lst[:],
        lst.copy(),
        [*lst]]
for i, v in enumerate(llst):
    v[2].append("#{}".format(i))
print(lst)

d = {"a": {"b": [0]}}
dd = [d,
      d.copy(),
      {**d}]
for i, v in enumerate(dd):
    v['a']['b'].append("#{}".format(i))
print(d)
```

    [0, 1, [2, 3, '#0', '#1', '#2', '#3']]
    {'a': {'b': [0, '#0', '#1', '#2']}}


这些 copy 的方法称为**浅拷贝（shallow copy）**，它相比直接赋值更进了一步生成了新的对象，但是对于嵌套的对象仍然采用了赋值的方法来创建；如果要再进一步，则需要**深拷贝（deep copy）**，由标准库 `copy` 提供：


```python
from copy import deepcopy

lst = [0, 1, [2, 3]]
lst2 = deepcopy(lst)
lst2[2].append(4)
print(lst2)
print(lst)

d = {"a": {"b": [0]}}
d2 = deepcopy(d)
d2["a"]["b"].append(1)
print(d2)
print(d)
```

    [0, 1, [2, 3, 4]]
    [0, 1, [2, 3]]
    {'a': {'b': [0, 1]}}
    {'a': {'b': [0]}}


清楚了赋值（引用）、copy 还是 `deepcopy` 之间的区别才能更好地避免意想不到的错误，同样也可以利用它们的特性去实现一些 little tricks，例如我们在 [0x04 闭包与作用域](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-10-Scope-and-Closure.ipynb) 中利用可变对象的特性实现 `nonlocal` 的功能。关于可变对象的引用、传递等既是 Python 的基本属性，同时又因为隐藏在背后的“暗箱操作”而容易引起误解，想要深入了解可以进一步阅读参考链接的文章，我也会在后面的文章中继续一边学习、一边补充更多这方面的知识。

### 参考

1. [python基础（5）：深入理解 python 中的赋值、引用、拷贝、作用域](http://my.oschina.net/leejun2005/blog/145911)
