title: 常见文件编码与unicode
date: 2017-05-15 21:13:04
tags:
- unicode
- utf8
- decoding
- gbk
- utf16
---


编码问题大概是经常处理各种格式的输入输出最容易遇到，也最让人头疼的错误，即使小心设定编码，也很容易在各种各样地方踩坑。

于是打算从编码的规范本身进行一些归类和梳理，方便更好的理解python里面decode和encode，以及unicode的概念。

## 1.ASCII和非ASCII编码
ASCII是最基础的文件显示编码，它只有8位长度，只能表示256个字符。
固定存储为一个字节。
后来ASCII码还进行了一些扩展，用于显示拉丁语系的一些特殊字符。EASCII和Latin-1都是ASCII码的扩展。

<!-- more -->

ASCII是有美国订立的，主要也是服务英文国家。

为了显示非英文字符，不同的国家和地区制定了不同的标准，由此产生了 GB2312、GBK、Big5、Shift\_JIS 等各自的编码标准。这些使用 1 至 4 个字节来代表一个字符的各种汉字延伸编码方式，统称为 ANSI 编码。在简体中文Windows操作系统中，ANSI 编码代表 GBK 编码；在日文Windows操作系统中，ANSI 编码代表 Shift\_JIS 编码。 不同 ANSI 编码之间互不兼容，当信息在国际间交流时，无法将属于两种语言的文字，存储在同一段 ANSI 编码的文本中。

GB2312是一个典型的双字节编码，兼容ASCII。只表示英文的时候，使用单字节表示。所以早期接触计算机的同学应该还记得，英文一个字节，汉字两个字节，这样的说法。这种说法只针对于用GB2312/GBK编码的文件才有效.
实际编码只收录了6763个汉字，部分罕见字和繁体字都无法显示。后来被扩展的GBK所取代。
  
最新的中文编码国标标准GB18030，则是为了和UNICODE规范靠拢的一个中文编码，是一种变长的编码。在单字节和双字节分别兼容ASCII和GBK编码，而显示其他UNICODE标准字符则使用4字节编码显示。

那么UNICODE究竟是什么呢？

## 2.UNICODE编码规范
  
  unicode并不是一种编码格式，而是一个编码规范，也就是一个文档标准。
  
  简单来说就是标准化组织提供的一个编码表，然后大家都是用这个编码表来对字符进行编码。
  unicode的编码表一共分成17组编排，每组称为平面（Plane），而每平面拥有256*256个字符。
  每个Plane的值为 0000到FFFF
  第N格平面则再最高位加1
  比如1号 Plane则编码为 10000 – 1FFFF
  其中0号Plane加上1号Plane已经包含大部分的世界语言文字. 
  https://zh.wikipedia.org/wiki/Unicode%E5%AD%97%E7%AC%A6%E5%B9%B3%E9%9D%A2%E6%98%A0%E5%B0%84

  那么规范有了，具体实现则是又有各种各样的实现。这种实现就叫做Unicode转换格式（Unicode Transformation Format，简称为UTF）。
  因此UTF-8，UTF-16 LE，UTF-16 BE， UTF-32都是一种unicode转换格式，是实际上在操作系统上使用的unicode文件编码。
  
  在UNICODE规范中, 整个16进制编码范围从U+00000到U+10FFFF，需要至少21位来表示.最简单的想法那就用这21位直接表示对应的文字,弄个 UTF-21就好了,然而实际上并没有这样做,让我们来看看是为什么呢.

  即使把中日韩的语言考虑在内,大部分情况下使用的文字只是用到了unicode 规范中的前两个 Plane, 也就是基本平面和基本扩展平面.从0000-1FFFF. 如果每次都使用完全的位表示,会有很多冗余的0,导致存储空间的浪费.
  所以 utf 编码都是使用了变长的编码技术.只有在显示其他平面编码时才会增加长度来显示,另外为了表示该字节是属于前一个字的还需要一些额外的位来表示该位是否是上一个字的变长编码段.

## 3. 实际编码实现

### UTF-8

以 UTF-8为例子. UTF-8是一种最常见的utf系列编码。
  它的编码规则为:
  
| 码点的位数 | 码点起值 | 码点终值 | 字节序列 | Byte 1 | Byte 2 | Byte 3 | Byte 4|
| -------- | ------  | ------ | ------ | ------ | ------ | ------ | ------| 
| 7  |U+0000  |U+007F   |1 |0xxxxxxx| | | |
|11  |U+0080  |U+07FF   |2 |110xxxxx|  10xxxxxx| | |
|16  |U+0800  |U+FFFF   |3 |1110xxxx|  10xxxxxx|  10xxxxxx| |
|21  |U+10000 |U+1FFFFF |4 |11110xxx|  10xxxxxx|  10xxxxxx|  10xxxxxx|


对于7位以下的编码,采用首位填0,后面七位作为编码的实现,和 ASCII 编码完全兼容.
对于7FF 以下的编码则使用两个字节来编码,可以查询前面的 Unicode 编码表格,这里主要是拉丁语和阿拉伯语以及国际音标等编码.
对于剩下的第0平面的文字则使用三个字节,所以大部分中文的utf8编码都是三个字节.也有少数生僻字不在第0平面,则用 utf-8编码是4个字节.

UTF-8编码与其他编码设计上还有一个自解释的效果.
第一个字节里面的1个数表示总编码长度. 10开头表示是中间字节,存储编码和占位编码之间用0分割.
所以在 utf-8里是不会有两个连续11开头的字节在存在的.也不会有 FE 和 FF 这两个编码.

### UTF-16

UTF-16是另一个常用的 unicode编码. 在 windows 中使用比较多.

它的编码规则为:

|16进制编码范围  |UTF-16表示方法（二进制）| 10进制码范围 | 字节数量|
| -----        | -----              | -----      | ----- |
|U+0000---U+FFFF |xxxxxxxx xxxxxxxx yyyyyyyy yyyyyyyy| 0-65535| 2|
|U+10000---U+10FFFF | 110110yyyyyyyyyy 110111xxxxxxxxxx |65536-1114111| 4|

会经过计算把 Unicode 编码分成高位和低位,然后再按照表格处理.
UTF-16 对于英文也编码成了两个字节,并不兼容 ASCII, 但是对比 utf-8,在存储中文的时候一般只需要两个字节,空间上还是有节省的.

对于高位在前还是低位在前, 根据实现不同也分为 UTF-16-LE 和 UTF-16-BE
window和 linux 使用 UTF-16-LE, 而 mac 使用UTF-16-BE

因此 UTF-16的文件开头会有 BOM ( 两位的字符FE FF或者 FF FE)表示该文件是 LE 还是 BE 的.

有时候有些编辑器也会给 UTF-8带上 BOM, 然而实际上 UTF-8是不需要 BOM 的.有可能就会导致出错。

再简单说一下我们的国标.

### GB18030

中文 GB18030是实现了中文的 Unicode 标准的编码.而 GB2312/GBK 则是之前的非 unicode旧标准.

中文的 GB18030在扩展平面,也就是4字节的区域, 也使用了 UTF-16的类似定义.而对于U+0000-U+FFFF  GB18030是兼容 ASCii 编码,兼容 GBK 和 GB2312.有自己的独特实现.

说了一堆，看下python里面的例子就更容易清楚了。

```
>>> a = "中文"
>>> a
'\xe4\xb8\xad\xe6\x96\x87'
>>> a.decode('utf8')
u'\u4e2d\u6587'
>>> a.decode('utf8').encode('gbk')
'\xd6\xd0\xce\xc4'
```

中文对应的UNICODE码表为 u4e2d 和 u6587，编码成utf8之后每个中文对应三个字节，\xe4\xb8\xad \xe6\x96\x87, 编码成GBK每个中文对应2个字节 \xd6\xd0\xce\xc4.而每个编码和原来的UNICODE都不一样，所以是一种一对一映射。

```
>>> a = 'a'
>>> a.encode('utf-16')
'\xff\xfea\x00'
上面的输出加了 BOM
>>> u'a'.encode('utf-16-le')
'a\x00'
>>> a.encode('utf-16-be')
'\x00a'
```

对于一个大小超过U+FFFF的unicode编码，都编码为4个
```
>>> u'\U0003FFFF'.encode('utf-16-be')
'\xd8\xbf\xdf\xff'
>>> u'\U0003FFFF'.encode('utf-16-le')
'\xbf\xd8\xff\xdf'
>>> u'\U0003FFFF'.encode('gb18030')
'\x9f6\x857'
>>> u'\U0003FFFF'.encode('utf-8')
'\xf0\xbf\xbf\xbf'
```

### 4. 小结

在python2中，当str的编码不同时，转换编码会使用unicode来作为中间代码来处理各种编码。python默认decode会使用系统环境的默认编码来进行解码，如果解码失败就会抛出异常。

而在python文件开头的注释-*- coding:xxx -*- 则是明确告诉python当前文件代码中的使用何种编码

统一在程序所有接口使用同一种编码是减少出错的最佳方法。

参考资料:

[http://www.voidcn.com/blog/haiross/article/p-3165405.html](http://www.voidcn.com/blog/haiross/article/p-3165405.html)

[http://scripts.sil.org/cms/scripts/page.php?site_id=nrsi&item_id=IWS-Chapter03](http://scripts.sil.org/cms/scripts/page.php?site_id=nrsi&item_id=IWS-Chapter03)

[http://www.cnblogs.com/lislok/archive/2008/10/14/1311041.html](http://www.cnblogs.com/lislok/archive/2008/10/14/1311041.html)

[http://www.cnblogs.com/gavin-num1/p/5170247.html](http://www.cnblogs.com/gavin-num1/p/5170247.html)
