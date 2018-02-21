---
title: python的collections库简介
date: 2018-02-21 16:01:19
tags:
- python
- 数据结构
---

list，dict这类是我们在python中最常用数据容器。

然而这两个类型并不总是完全够用，我们常常在编程中会遇到的一个情况就是如何给字典的key排序。

当然可以自己实现一个dict的继承或者实现一个对key进行排序的函数，不过标准库里面已经有了这样的实现。

这就是collections库。

python的标准库中collections定义了一些常用的容器类型。

包括命名元祖namedtuple，双向队列deque，计数器Counter，有序字典OrderedDict，默认值字典defaultdict。

<!-- more -->

## 1.deque

虽然提供和list同样的操作，但是对于在头尾增加元素进行了优化。随机读取的效率不如list。

猜测这个数据结构的内部实现应该是使用一个头尾指针的链表制作的。

去对比一下list的源码

```
>>> from collections import deque
>>> d = deque('ghi')                 # make a new deque with three items
>>> for elem in d:                   # iterate over the deque's elements
...     print elem.upper()
G
H
I

>>> d.append('j')                    # add a new entry to the right side
>>> d.appendleft('f')                # add a new entry to the left side
>>> d                                # show the representation of the deque
deque(['f', 'g', 'h', 'i', 'j'])
```

## 2.Counter

Counter是继承至dict，构造函数是传入一个可迭代的对象，计算出其中重复元素的个数，重复元素将作为key，而计数将作为结果存储。

使用update来更新计数。

most_common()可以查看前N项计数

```
>>> # Tally occurrences of words in a list
>>> cnt = Counter()
>>> for word in ['red', 'blue', 'red', 'green', 'blue', 'blue']:
...     cnt[word] += 1
>>> cnt
Counter({'blue': 3, 'red': 2, 'green': 1})

>>> # Find the ten most common words in Hamlet
>>> import re
>>> words = re.findall(r'\w+', open('hamlet.txt').read().lower())
>>> Counter(words).most_common(10)
[('the', 1143), ('and', 966), ('to', 762), ('of', 669), ('i', 631),
 ('you', 554),  ('a', 546), ('my', 514), ('hamlet', 471), ('in', 451)]
```

## 3.OrderedDict

有序字典，构造函数和dict一样，传入一个二元组的list。

字典的迭代顺序将会按照插入字典的顺序来迭代。

不支持生成后，对OrderDict的重排序。如果需要只能够生成另外一个OrderDict。可能造成额外的空间开销。

```
>>> # regular unsorted dictionary
>>> d = {'banana': 3, 'apple': 4, 'pear': 1, 'orange': 2}

>>> # dictionary sorted by key
>>> OrderedDict(sorted(d.items(), key=lambda t: t[0]))
OrderedDict([('apple', 4), ('banana', 3), ('orange', 2), ('pear', 1)])
```

## 4.namedtuple

命名元组的概念和有序字典优点类似，都可以用key来访问的有序的容器，不过是不可变对象。

不过生成的对象还是一个元组，只是可以用key来对下标的别名进行访问。

另外nametuple允许重名。

collections提供的Namedtuple返回的不是一个对象，而是一个class。是一个factory函数，生成一个Namedtuple类型。

除了用key访问也可以用下标访问，表现和元组一样

```
>>> Point = namedtuple('Point', ['x', 'y'], verbose=True)
>>> p = Point(11, y=22)     # instantiate with positional or keyword arguments
>>> p[0] + p[1]             # indexable like the plain tuple (11, 22)
33
>>> x, y = p                # unpack like a regular tuple
>>> x, y
(11, 22)
>>> p.x + p.y               # fields also accessible by name
33
>>> p                       # readable __repr__ with a name=value style
Point(x=11, y=22)

```

## 5.其他基类

collections还提供了一些使用的容器基类， 可以查阅文档

https://docs.python.org/2/library/collections.html#collections-abstract-base-classes
