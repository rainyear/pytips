
### 字节与字节数组

[0x07](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-15-Unicode-String.ipynb) 中介绍了 Python 中的字符串类型，字符串类型是对人类友好的符号，但计算机只认识一种符号，那就是二进制（binary）数，或者说是数字：

![OpenCV](http://docs.opencv.org/2.4/_images/MatBasicImageForComputer.jpg)

上面这张图片来自 [OpenCV](http://docs.opencv.org/2.4/)，非常直观地解释了计算机处理的信息与我们看到的图像之间的关系。回到 Python 对字节和字节数组的定义：

> The core built-in types for manipulating binary data are `bytes` and `bytearray`.

### 1Byte of ASCII

为了用计算机可以理解的数字描述人类使用的字符，我们需要一张数字与字符对应的表。我们都知道在计算机中 `1 byte = 8bits`，可以存储 `0~255` 共256个值，也就是说 `1byte` 最多可以表示 256 个字符，在最初的计算机世界中，256 足以容纳所有大小写英文字母和 `0~9` 阿拉伯数字以及一些常用的符号，于是就有了 ASCII 编码：

![ascii](http://qncdn.rainy.im/asciix400.jpg)

在 Python 中创建字节与字符串类似，只不过需要在引号外面加一个前缀`b`：


```python
print(b"Python")
python = (b'P' b'y' b"t" b'o' b'n')
print(python)
```

    b'Python'
    b'Pyton'


Bytes 代表的是（二进制）数字的序列，只不过在是通过 `ASCII` 编码之后才是我们看到的字符形式，如果我们单独取出一个字节，它仍然是一个数字：


```python
print(b"Python"[0])
```

    80


我们可以用 `b"*"` 的形式创建一个字节类型，前提条件是这里的 `*` 必须是 `ASCII` 中可用的字符，否则将会超出限制：


```python
print(b"雨")
```


      File "<ipython-input-3-1f95d71c6553>", line 1
        print(b"雨")
             ^
    SyntaxError: bytes can only contain ASCII literal characters.



错误提示说明：字节类型只能允许 ASCII 字符。

**0~127~255**

那么问题来了，我们发现上面的 `ASCII` 表里面所有的字符只占据了 `[31, 127]`，那对于这一范围之外的数字我们要怎么才能表示为字节类型？答案就是用特殊的转义符号`\x`+十六进制数字 ：


```python
print(b'\xff'[0])
print(b'\x24')
```

    255
    b'$'


反过来我们也可以将数字（0~255）转变成转义后的字节类型：


```python
print(bytes([24]))
print(bytes([36,36,36])) # 记住字节类型是一个序列
```

    b'\x18'
    b'$$$'


或者直接从十六进制得来：


```python
print(bytes.fromhex("7b 7d"))

# 逆运算
print(b'{ }'.hex())

int(b' '.hex(), base=16)
```

    b'{}'
    7b207d





    32



### `encode`

字符串有 `decode` 方法，而字节有 `encode` 方法，我们这里先简单看一下 `encode('ascii')` 。对于给定的**字符**我们可以通过编码得到它在编码表里面的坐标（即码位），因此对字符进行`encode('ascii')`操作是找到其在 `ASCII` 中的位置：


```python
print("$".encode('ascii'))
print("$".encode('ascii')[0])
```

    b'$'
    36


也就是说字符 `"$"` （[0x07](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-15-Unicode-String.ipynb)中已经介绍过这是一个 Unicode 编码的字符）在 `ASCII` 中的位置就是 `$`（或者说36）。

可是如果我们对一些奇怪的字符进行 `ASCII` 编码，就会发生：


```python
snake = '🐍'
try:
    snake.encode('ascii')
except UnicodeEncodeError as err:
    print(err)

# 正确的做法应该是用 UTF-8 进行编码，因为字符串都是 UTF-8 的
print(snake.encode()) # utf-8 by default
```

    'ascii' codec can't encode character '\U0001f40d' in position 0: ordinal not in range(128)
    b'\xf0\x9f\x90\x8d'


于是就得到了我们最熟悉的错误：`ordinal not in range(128)`，至于为什么是 128，现在应该很好理解了吧！

### 字节数组

和字符串一样，字节类型也是不可变序列，而字节数组就是可变版本的字节，它们的关系就相当于`list`与`tuple`。


```python
ba = bytearray(b'hello')
ba[0:1] = b'w'
print(ba)
```

    bytearray(b'wello')


由于和字符串一样是序列类型，字节和字节数组可用的方法也类似，这里就不一一列举了。

### 总结

1. 字节（字节数组）是二进制数据组成的序列，其中每个元素由8bit二进制即1byte亦即2位十六进制数亦亦即0~255组成；
2. 字节是计算机的语言，字符串是人类语言，它们之间通过编码表形成一一对应的关系；
3. 最小的 `ASCII` 编码表只需要一位字节，且只占用了其中 `[31,127]` 的码位；

关于字节与字符串之间的关系，将在下一期[0x08]()详细介绍。

### 参考

1. [Pragmatic Unicode](http://nedbatchelder.com/text/unipain/unipain.html#1)
