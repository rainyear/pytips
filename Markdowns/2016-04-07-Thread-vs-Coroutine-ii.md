
### Python 线程与协程（2）

我之前翻译了[Python 3.5 协程原理](http://blog.rainy.im/2016/03/10/how-the-heck-does-async-await-work-in-python-3-5/)这篇文章之后尝试用了 [Tornado + Motor](https://github.com/rainyear/lolita/issues/28) 模式下的协程进行异步开发，确实感受到协程所带来的好处（至少是语法上的`:D`）。至于协程的 `async/await` 语法是如何由开始的 `yield` 生成器一步一步上位至 Python 的 `async/await` 组合语句，前面那篇翻译的文章里面讲得已经非常详尽了。我们知道协程的本质上是：

> allowing multiple entry points for suspending and resuming execution at certain locations.

允许多个入口对程序进行挂起、继续执行等操作，我们首先想到的自然也是生成器：


```python
def jump_range(upper):
    index = 0
    while index < upper:
        jump = yield index
        if jump is None:
            jump = 1
        index += jump
jump = jump_range(5)
print(jump)
print(jump.send(None))
print(jump.send(3))
print(jump.send(None))
```

    <generator object jump_range at 0x10e283518>
    0
    3
    4


后来又新增了 `yield from` 语法，可以将生成器串联起来：


```python
def wait_index(i):
    # processing i...
    return (yield i)
def jump_range(upper):
    index = 0
    while index < upper:
        jump = yield from wait_index(index)
        if jump is None:
            jump = 1
        index += jump
jump = jump_range(5)
print(jump)
print(jump.send(None))
print(jump.send(3))
print(jump.send(None))
```

    <generator object jump_range at 0x10e22a780>
    0
    3
    4


`yield from`/`send` 似乎已经满足了协程所定义的需求，最初也确实是用 [`@types.coroutine` 修饰器](https://hg.python.org/cpython/file/3.5/Lib/types.py#l206)将生成器转换成协程来使用，在 Python 3.5 之后则以专用的 `async/await` 取代了 `@types.coroutine/yield from`：


```python
class Wait(object):
    """
    由于 Coroutine 协议规定 await 后只能跟 awaitable 对象，
    而 awaitable 对象必须是实现了 __await__ 方法且返回迭代器
    或者也是一个协程对象，
    因此这里临时实现一个 awaitable 对象。
    """
    def __init__(self, index):
        self.index = index
    def __await__(self):
        return (yield self.index)
async def jump_range(upper):
    index = 0
    while index < upper:
        jump = await Wait(index)
        if jump is None:
            jump = 1
        index += jump
jump = jump_range(5)
print(jump)
print(jump.send(None))
print(jump.send(3))
print(jump.send(None))
```

    <coroutine object jump_range at 0x10e2837d8>
    0
    3
    4


**与线程相比**

协程的执行过程如下所示：


```python
import asyncio
import time
import types

@types.coroutine
def _sum(x, y):
    print("Compute {} + {}...".format(x, y))
    yield time.sleep(2.0)
    return x+y
@types.coroutine
def compute_sum(x, y):
    result = yield from _sum(x, y)
    print("{} + {} = {}".format(x, y, result))
loop = asyncio.get_event_loop()
loop.run_until_complete(compute_sum(0,0))
```

    Compute 0 + 0...
    0 + 0 = 0


![tulip_coro](http://qncdn.rainy.im/tulip_coro.png)

这张图（来自: [PyDocs: 18.5.3. Tasks and coroutines](https://docs.python.org/3/library/asyncio-task.html)）清楚地描绘了由事件循环调度的协程的执行过程，上面的例子中事件循环的队列里只有一个协程，如果要与上一部分中线程实现的并发的例子相比较，只要向事件循环的任务队列中添加协程即可：


```python
import asyncio
import time

# 上面的例子为了从生成器过度，下面全部改用 async/await 语法
async def _sum(x, y):
    print("Compute {} + {}...".format(x, y))
    await asyncio.sleep(2.0)
    return x+y
async def compute_sum(x, y):
    result = await _sum(x, y)
    print("{} + {} = {}".format(x, y, result))

start = time.time()
loop = asyncio.get_event_loop()
tasks = [
    asyncio.ensure_future(compute_sum(0, 0)),
    asyncio.ensure_future(compute_sum(1, 1)),
    asyncio.ensure_future(compute_sum(2, 2)),
]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
print("Total elapsed time {}".format(time.time() - start))
```

    Compute 0 + 0...
    Compute 1 + 1...
    Compute 2 + 2...
    0 + 0 = 0
    1 + 1 = 2
    2 + 2 = 4
    Total elapsed time 2.0042951107025146


### 总结

这两篇主要关于 Python 中的线程与协程的一些基本原理与用法，为此我搜索了不少参考文章与链接，对我自己理解它们的原理与应用场景也有很大的帮助（当然也有可能存在理解不到位的地方，欢迎指正）。当然在这里还是主要关注基于 Python 的语法与应用，如果想要了解更多底层实现的细节，可能需要从系统调度等底层技术细节开始学习（几年前我记得翻阅过《深入理解LINUX内核》这本书，虽然大部分细节已经记不清楚了，但对于理解其它人的分析、总结还是有一定帮助的）。这里讨论的基于协程的异步主要是借助于事件循环（由`asyncio`标准库提供），包括上文中的示意图，看起来很容易让人联想到 `Node.js` 的事件循环 & 回调，但是协程与回调也还是有区别的，具体就不在这里展开了，可以参考下面第一条参考链接。

### 参考

1. [Python 中的进程、线程、协程、同步、异步、回调](https://segmentfault.com/a/1190000001813992)
2. [我是一个线程](http://mp.weixin.qq.com/s?__biz=MjM5NzA1MTcyMA==&mid=403498894&idx=2&sn=219c1a6001b5bb7e6bdc7963b1af8450)
3. [Concurrency is not Parallelism](https://talks.golang.org/2012/waza.slide#1)
4. [A Curious Course on Coroutines and Concurrency](http://www.dabeaz.com/coroutines/Coroutines.pdf)
5. [PyDocs: 17.1. threading — Thread-based parallelism](https://docs.python.org/3.5/library/threading.html)
6. [PyDocs: 18.5.3. Tasks and coroutines](https://docs.python.org/3/library/asyncio-task.html)
7. [[译] Python 3.5 协程究竟是个啥](http://blog.rainy.im/2016/03/10/how-the-heck-does-async-await-work-in-python-3-5/)
8. [协程的好处是什么？ - crazybie 的回答](https://www.zhihu.com/question/20511233/answer/83307265)
9. [Py3-cookbook:第十二章：并发编程](http://python3-cookbook.readthedocs.org/zh_CN/latest/chapters/p12_concurrency.html)
10. [Quora: What are the differences between parallel, concurrent and asynchronous programming?](https://www.quora.com/What-are-the-differences-between-parallel-concurrent-and-asynchronous-programming)
11. [Real-time apps with gevent-socketio](http://learn-gevent-socketio.readthedocs.org/en/latest/index.html)
