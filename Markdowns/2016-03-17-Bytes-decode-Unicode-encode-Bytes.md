
## Python 中 Unicode 的正确用法

[0x07](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-15-Unicode-String.ipynb) 和 [0x08](https://github.com/rainyear/pytips/blob/master/Tips/2016-03-16-Bytes-and-Bytearray.ipynb) 分别介绍了 Python 中的字符串类型（`str`）和字节类型（`byte`），以及 Python 编码中最常见也是最顽固的两个错误：

> UnicodeEncodeError: 'ascii' codec can't encode characters in position 0-1: ordinal not in range(128)

> UnicodeDecodeError: 'utf-8' codec can't decode bytes in position 0-1: invalid continuation byte

这一期就从这两个错误入手，分析 Python 中 Unicode 的正确用法。这篇短文并不能保证你可以永远杜绝上面两个错误，但是希望在下次遇到这个错误的时候知道错在哪里、应该从哪里入手。

### 编码与解码

上面的两个错误分别是 `UnicodeEncodeError` 和 `UnicodeDecodeError`，也就是说分别在 Unicode 编码（Encode）和解码（Decode）过程中出现了错误，那么编码和解码究竟分别意味着什么？根据维基百科[字符编码](https://zh.wikipedia.org/wiki/字符编码)的定义：

> 字符编码（英语：Character encoding）、字集码是把字符集中的字符编码为指定集合中某一对象（例如：比特模式、自然数序列、8位组或者电脉冲），以便文本在计算机中存储和通过通信网络的传递。

简单来说就是把**人类通用的语言符号**翻译成**计算机通用的对象**，而反向的翻译过程自然就是**解码**了。Python 中的字符串类型代表人类通用的语言符号，因此字符串类型有`encode()`方法；而字节类型代表计算机通用的对象（二进制数据），因此字节类型有`decode()`方法。


```python
print("🌎🌏".encode())
```

    b'\xf0\x9f\x8c\x8e\xf0\x9f\x8c\x8f'



```python
print(b'\xf0\x9f\x8c\x8e\xf0\x9f\x8c\x8f'.decode())
```

    🌎🌏


既然说编码和解码都是**翻译**的过程，那么就需要一本字典将人类和计算机的语言一一对应起来，这本字典的名字叫做**字符集**，从最早的 ASCII 到现在最通用的 Unicode，它们的本质是一样的，只是两本字典的厚度不同而已。ASCII 只包含了26个基本拉丁字母、阿拉伯数目字和英式标点符号一共128个字符，因此只需要（不占满）一个字节就可以存储，而 Unicode 则涵盖的数据除了视觉上的字形、编码方法、标准的字符编码外，还包含了字符特性，如大小写字母，共可包含 1.1M 个字符，而到现在只填充了其中的 110K 个位置。

字符集中字符所存储的位置（或者说对应的计算机通用的数字）称之为码位（code point），例如在 ASCII 中字符 `'$'` 的码位就是：


```python
print(ord('$'))
```

    36


ASCII 只需要一个字节就能存下所有码位，而 Unicode 则需要几个字节才能容纳，但是对于具体采用什么样的方案来实现 Unicode 的这种映射关系，也有很多不同的方案（或规则），例如最常见（也是 Python 中默认的）UTF-8，还有 UTF-16、UTF-32 等，对于它们规则上的不同这里就不深入展开了。当然，在 ASCII 与 Unicode 之间还有很多其他的字符集与编码方案，例如中文编码的 GB2312、繁体字的 Big5 等等，这并不影响我们对编码与解码过程的理解。

### Unicode\*Error

明白了字符串与字节，编码与解码之后，让我们手动制造上面两个 `Unicode*Error` 试试，首先是编码错误：


```python
def tryEncode(s, encoding="utf-8"):
    try:
        print(s.encode(encoding))
    except UnicodeEncodeError as err:
        print(err)
    
s = "$"           # UTF-8 String
tryEncode(s)          # 默认用 UTF-8 进行编码
tryEncode(s, "ascii") # 尝试用 ASCII 进行编码

s = "雨"          # UTF-8 String
tryEncode(s)          # 默认用 UTF-8 进行编码
tryEncode(s, "ascii") # 尝试用 ASCII 进行编码
tryEncode(s, "GB2312")  # 尝试用 GB2312 进行编码
```

    b'$'
    b'$'
    b'\xe9\x9b\xa8'
    'ascii' codec can't encode character '\u96e8' in position 0: ordinal not in range(128)
    b'\xd3\xea'


由于 UTF-8 对 ASCII 的兼容性，`"$"` 可以用 ASCII 进行编码；而 `"雨"` 则无法用 ASCII 进行编码，因为它已经超出了 ASCII 字符集的 128 个字符，所以引发了 `UnicodeEncodeError`；而 `"雨"` 在 GB2312 中的码位是 `b'\xd3\xea'`，与 UTF-8 不同，但是仍然可以正确编码。因此如果出现了 `UnicodeEncodeError` 说明你用错了字典，要翻译的字符没办法正确翻译成码位！

再来看解码错误：


```python
def tryDecode(s, decoding="utf-8"):
    try:
        print(s.decode(decoding))
    except UnicodeDecodeError as err:
        print(err)
        
b = b'$'     # Bytes
tryDecode(b)          # 默认用 UTF-8 进行解码
tryDecode(b, "ascii") # 尝试用 ASCII 进行解码
tryDecode(b, "GB2312") # 尝试用 GB2312 进行解码

b = b'\xd3\xea' # 上面例子中通过 GB2312 编码得到的 Bytes
tryDecode(b)           # 默认用 UTF-8 进行解码
tryDecode(b, "ascii")  # 尝试用 ASCII 进行解码
tryDecode(b, "GB2312") # 尝试用 GB2312 进行解码
tryDecode(b, "GBK")    # 尝试用 GBK 进行解码
tryDecode(b, "Big5")    # 尝试用 Big5 进行解码

tryDecode(b.decode("GB2312").encode()) # Byte-Decode-Unicode-Encode-Byte
```

    $
    $
    $
    'utf-8' codec can't decode byte 0xd3 in position 0: invalid continuation byte
    'ascii' codec can't decode byte 0xd3 in position 0: ordinal not in range(128)
    雨
    雨
    迾
    雨


一般后续出现的字符集都是对 ASCII 兼容的，可以认为 ASCII 是他们的一个子集，因此可以用 ASCII 进行解码（编码）的，一般也可以用其它方法；对于不是不存在子集关系的编码，强行解码有可能会导致错误或乱码！

### 实践中的策略

清楚了上面介绍的所有原理之后，在时间操作中应该怎样规避错误或乱码呢？

1. 记清楚编码与解码的方向；
2. 在 Python 中的操作尽量采用 UTF-8，输入或输出的时候再根据需求确定是否需要编码成二进制：


```python
# cat utf8.txt
# 你好，世界！
# file utf8.txt
# utf8.txt: UTF-8 Unicode text

with open("utf8.txt", "rb") as f:
    content = f.read()
    print(content)
    print(content.decode())
with open("utf8.txt", "r") as f:
    print(f.read())
    
# cat gb2312.txt
# 你好，Unicode！
# file gb2312.txt
# gb2312.txt: ISO-8859 text

with open("gb2312.txt", "r") as f:
    try:
        print(f.read())
    except:
        print("Failed to decode file!")
with open("gb2312.txt", "rb") as f:
    print(f.read().decode("gb2312"))
```

    b'\xe4\xbd\xa0\xe5\xa5\xbd\xef\xbc\x8c\xe4\xb8\x96\xe7\x95\x8c\xef\xbc\x81\n'
    你好，世界！
    
    你好，世界！
    
    Failed to decode file!
    你好，Unicode！
    


![Unicode](http://7xiijd.com1.z0.glb.clouddn.com/Pragmatic_Unicode.jpg)

### 参考

1. [Pragmatic Unicode](http://nedbatchelder.com/text/unipain/unipain.html)
2. [字符编码笔记：ASCII，Unicode和UTF-8](http://www.ruanyifeng.com/blog/2007/10/ascii_unicode_and_utf-8.html)
