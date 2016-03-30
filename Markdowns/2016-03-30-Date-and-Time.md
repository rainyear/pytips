
### Python 时间与日期

时间和日期可能涉及到不同的时区、格式，同时又经常需要作为时间戳保存，有时候还需要进行一些加减操作，因此处理起来通常会因为方法太多而无从下手。Python 中与时间和日期相关的标准库有3个：`time`、`datetime` 和 `calendar`。其中 `time` 模块更偏向于系统相关的时间数据，最常用的可能就是获取当前时间的秒数 `time.time()`，此外该模块中的很多方法都是与 C 语言中的时间方法相同的，如果习惯了使用 C 的人可以很方便地继续使用这些方法。`calendar` 则是在时间与日期之上，它的作用真的就是“日历”：


```python
from calendar import TextCalendar, HTMLCalendar

tc = TextCalendar(firstweekday=6)
tc.prmonth(2016, 3)
```

         March 2016
    Su Mo Tu We Th Fr Sa
           1  2  3  4  5
     6  7  8  9 10 11 12
    13 14 15 16 17 18 19
    20 21 22 23 24 25 26
    27 28 29 30 31
     

`datetime` 模块解决了绝大部分时间与日期相关的操作问题，其中包含了：

1. `timedelta` 与时间计算相关的类；
3. `time` 时间相关的类；
4. `date` 日期相关的类；
5. `datetime` 时间和日期；
2. `tzinfo`/`timezone` 与时区相关的类（`timezone` 是 Python 3.2 之后新加入的）；

它们的继承关系如下：


```python
"""
object
    timedelta
    tzinfo
        timezone
    time
    date
        datetime
"""
pass
```


```python
import time as _time
from datetime import date, time, datetime
```

### Date

`date` 由年、月、日组成，有下面几种方式创建一个 `date` 对象（`strptime` 是通用方法，将在后面介绍）：


```python
d1 = date(2016, 3, 29)
d2 = date.today()
d3 = date.fromtimestamp(_time.time())

print(d1)
print(d2)
print(d3)
```

    2016-03-29
    2016-03-30
    2016-03-30


获得 `date` 对象之后，可以分别获取年、月、日等属性（`strftime`也是通用的格式化方法，将在后面介绍）：


```python
print("{}/{}/{}".format(d2.day, d2.month, d2.year))

# date.timetuple() 返回 time 模块中的 struct_time 结构，可以直接转换成 Tuple
print("time.struct_time: {}".format(tuple(d2.timetuple()))) 

# 星期数
print("Monday is 0: {}\nMonday is 1: {}".format(d2.weekday(), d2.isoweekday()))
```

    30/3/2016
    time.struct_time: (2016, 3, 30, 0, 0, 0, 2, 90, -1)
    Monday is 0: 2
    Monday is 1: 3


### Time

`datetime.time` 由小时、分钟、秒、微秒（百万分之一秒）组成，和 `date` 相似的创建方式，但是由于和 `time` 模块有所重叠，因此并不经常直接用于创建时间对象，如果需要可以从 `datetime.datetime` 分割出来：


```python
t1 = time(22, 57, 6, 6)
t2 = datetime.now().time()
print(t1)
print(t2)
```

    22:57:06.000006
    23:56:12.495372


### datetime.datetime

`datetime.datetime` 继承自 `date`，同时将精度精确到时间，创建方式有：


```python
dt1 = datetime(2016, 3, 30, 22, 2)
dt2 = datetime.now()
dt3 = datetime.fromtimestamp(_time.time())
print(dt1)
print(dt2)
print(dt3)
```

    2016-03-30 22:02:00
    2016-03-30 23:56:13.800861
    2016-03-30 23:56:13.800924


从 `datetime.datetime` 中我们可以获取 `date` 和 `time`，同样也可以通过 `date` 和 `time` 组合得来：


```python
dt = datetime.now()
dt = datetime.fromtimestamp(_time.time())

d = dt.date()
t = dt.time()

print("Date: {}\nTime: {}".format(d, t))

print("Datetime: {}".format(datetime.combine(date.today(), time(2,3,3))))
```

    Date: 2016-03-30
    Time: 23:56:15.078349
    Datetime: 2016-03-30 02:03:03


### 时间与日期的运算

如果把 `date`、`time` 和 `datetime` 看作是时间轴上的点，那么 `timedelta` 就是时间轴上的线段（时间段，时间间隔）。


```python
from datetime import timedelta

td = timedelta(weeks=1, days=2, hours=3,minutes=4, seconds=0, microseconds=0, milliseconds=0)

print("Time duration: {}".format(td))
```

    Time duration: 9 days, 3:04:00


既然是时间段，那就可以通过两个时间点相减得到：


```python
current = datetime.now()
today = datetime.combine(date.today(), time(0,0,0))

td = current - today
print("{:.0f}s of Today".format(td.total_seconds()))

today = date.today()
lastyear = today.replace(year=today.year-1)
print(today - lastyear)

t1 = current.time()
t2 = time(0, 0, 0)
try:
    print(t1 - t2)
except TypeError as err:
    print(err)
```

    86178s of Today
    366 days, 0:00:00
    unsupported operand type(s) for -: 'datetime.time' and 'datetime.time'


时间段还支持一些算术`+`、`-`、`*`、`/`、`//`、`%`、`abs` 等，这里就不一一举例了。

### strftime & strptime

`strftime`（String from Time）和 `strptime`（String parsed Time）分别是字符串和时间日期之间的转换方法，只不过遵循一定的格式：


```python
print(datetime.strftime.__doc__)
print(datetime.strptime.__doc__)
```

    format -> strftime() style string.
    string, format -> new datetime parsed from a string (like time.strptime()).



```python
fmat = "%y-%m-%d"
dt = datetime.now()

s = dt.strftime(fmat)
print(s)
print(datetime.strptime(s, fmat))
```

    16-03-30
    2016-03-30 00:00:00


可以通过 [strftime() and strptime() Behavior](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior) 查看转换格式表，Python 使用的格式与 `C standard(1989)`是一致的，不过需要注意的是，所有格式都是 `zero-padded`，也就是自动补零的，如果想要去掉补零，可以用 `%-m` 等方式，但据说在 Windows 系统上是不能用的`:D`


```python
fmat = "%y/%-m/%-d"
dt = datetime.now()
dt = dt - timedelta(days=22)

print(dt.strftime(fmat))

# 当然也可以用
print("{}/{}/{}".format(dt.strftime("%y"), dt.month, dt.day))
```

    16/3/8
    16/3/8

