
### Python 的堆与优先队列

Python 中内置的 `heapq` 库和 `queue` 分别提供了堆和优先队列结构，其中优先队列 `queue.PriorityQueue` 本身也是基于 `heapq` 实现的，因此我们这次重点看一下 `heapq`。

堆（Heap）是一种特殊形式的完全二叉树，其中父节点的值总是大于子节点，根据其性质，Python 中可以用一个满足 `heap[k] <= heap[2*k+1] and heap[k] <= heap[2*k+2]` 的列表来实现（`heapq` 也确实是这么做的）。堆可以用于实现调度器（例见：[Python 3.5 之协程](http://blog.rainy.im/2016/03/10/how-the-heck-does-async-await-work-in-python-3-5/)），更常用的是优先队列（例如：[ImageColorTheme](https://github.com/rainyear/ImageColorTheme/blob/master/ict/MMCQ.py#L207)）。

`heapq` 提供了下面这些方法：


```python
import heapq
print(heapq.__all__)
```

    ['heappush', 'heappop', 'heapify', 'heapreplace', 'merge', 'nlargest', 'nsmallest', 'heappushpop']


由于 Heap 是通过列表实现的，我们可以直接用列表创建：


```python
from heapq import *
heap = []
heappush(heap, 3)
heappush(heap, 2)
heappush(heap, 1)
print(heap)
```

    [1, 3, 2]


#### pop 或 sort 前要确保 heapify

或者通过 `heapify` 将普通列表转化为 Heap：


```python
heap = list(reversed(range(5)))
print("List: ", heap)
heapify(heap)
print("Heap: ", heap)
```

    List:  [4, 3, 2, 1, 0]
    Heap:  [0, 1, 2, 4, 3]


每次从 Heap 中 `pop` 出来的元素都是最小的（因而可以据此实现堆排序）：


```python
heap = [5,4,3,2,1]
heapify(heap)
print(heappop(heap))
print(heappop(heap))
print(heappop(heap))
```

    1
    2
    3


#### 优先队列

`queue.PriorityQueue` 实际上只是对 `heapq` 的简单封装，直接使用其 `heappush`/`heappop` 方法：


```python
from queue import PriorityQueue as PQueue
pq = PQueue()
pq.put((5 * -1, 'Python'))
pq.put((4 * -1, 'C'))
pq.put((3 * -1, 'Js'))
print("Inside PriorityQueue: ", pq.queue) # 内部存储
while not pq.empty():
    print(pq.get()[1])
```

    Inside PriorityQueue:  [(-5, 'Python'), (-4, 'C'), (-3, 'Js')]
    Python
    C
    Js


由于 `heapq` 是最小堆，而通常 `PriorityQueue` 用在较大有限制的排前面，所以需要给 `priority * -1`。

#### sorted 一定是 Heap，反之未必

需要注意的是，虽然 Heap 通过 List 实习，但未经过 `heapify()` 处理的仍然是一个普通的 List，而 `heappush` 和 `heappop` 操作每次都会对 Heap 进行重新整理。此外，一个 Heap 列表不一定是正确排序的，但是经过 `list.sort()` 的列表一定是 Heap：


```python
import random
lst = [random.randrange(1, 100) for _ in range(5)]
lst.sort()
print("List: ", lst)
print("Poped: ", heappop(lst))
heappush(lst, 4)
print("Heap: ", lst)
```

    List:  [24, 55, 81, 83, 87]
    Poped:  24
    Heap:  [4, 55, 81, 87, 83]


#### 最大/最小的 N 个数

Heap 还提供了 `nsmallest` 和 `nlargest` 方法用于取出前 n 个最大/最小数：


```python
heap = [random.randrange(1, 1000) for _ in range(1000)]
heapify(heap)
print("N largest: ", nlargest(10, heap))
print("N smallest: ", nsmallest(10, heap))
print(len(heap))  # 不原地修改
```

    N largest:  [999, 999, 998, 994, 992, 991, 990, 988, 985, 982]
    N smallest:  [1, 1, 1, 2, 4, 5, 5, 6, 6, 9]
    1000


#### 合并（排序）

`merge` 方法用于将两个 Heap 进行合并：


```python
heapA = sorted([random.randrange(1, 100) for _ in range(3)])
heapB = sorted([random.randrange(1, 100) for _ in range(3)])

merged = []
for i in merge(heapA, heapB):
    merged.append(i)
print(merged)
```

    [5, 29, 66, 66, 70, 99]


最后两个方法 `heapreplace` 和 `heappushpop` 分别相当于：


```python
lstA = [1,2,3,4,5]
lstB = [1,2,3,4,5]

poped = heapreplace(lstA, 0)
print("lstA: ", lstA, "poped: ", poped)

# is equal to...
poped = heappop(lstB)
heappush(lstB, 0)
print("lstB: ", lstA, "poped: ", poped)

print("*"*30)

poped = heappushpop(lstA, 9)
print("lstA: ", lstA, "poped: ", poped)

# is equal to...
heappush(lstB, 9)
poped = heappop(lstB)
print("lstB: ", lstB, "poped: ", poped)
```

    lstA:  [0, 2, 3, 4, 5] poped:  1
    lstB:  [0, 2, 3, 4, 5] poped:  1
    ******************************
    lstA:  [2, 4, 3, 9, 5] poped:  0
    lstB:  [2, 4, 3, 5, 9] poped:  0


这两个方法的执行效率要比分开写的方法高，但要注意 `heapreplace` 要取代的值是否比 `heap[0]` 大，如果不是，可以用更有效的方法：


```python
item = 0
lstA = [1,2,3,4,5]
if item < lstA[0]:
    # replace
    poped = lstA[0]
    lstA[0] = item
    print("lstA: ", lstA, "poped: ", poped)
```

    lstA:  [0, 2, 3, 4, 5] poped:  1

