---
title: 如何发布自己制作的python库
date: 2018-02-13 23:43:10
tags:
- python
- pip
- setuptools
- disutil
---

python用了久了，很多人可能都或多或少写过自己的工具库。是否有想过，如果要把自己的python库上传到pypi要做哪些工作呢？

下面这篇文章就告诉大家要怎样开始。

<!-- more -->

## disutil

python文档中说明应当使用disutil来发布你的python package。

很多部分可以忽略，最重要的就是要有一个setup函数。



## setuptools

虽然不是官方库，已经成为了约定俗成的标准。大部分python库都是使用setuptools来进行安装和发布的。

而setuptools主要是对disutil标准库进行了一定的扩展。

setuptools也曾经有一个分支disutil2，后来这个分支并入了setuptools。

egg文件

eazyinstall.pth文件

pth文件

setup.py
setup.cfg
README.rst
MANIFEST.in
LICENSE.txt

## 上传到pypi

rst文件是目前pypi使用的一个文档描述文件

有些markdown工具可以转换到rst


## 私有地址

有时候，我们写的库可能只是随手写写，并不想扔到pypi上面去（面得代码太丑，贻笑大方）。

pip也可以使用自己的github地址或者gitlab地址来安装。




