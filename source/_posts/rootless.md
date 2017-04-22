---
title: rootless
date: 2017-04-21 10:06:21
tags:
- mac
- root
---

最近入手了一台新的macbook，升级了系统之后居然发现了一个诡异问题。

系统默认安装的six（python库）居然不能够升级，使用sudo（ All:（All，ALl））提示我权限不足。在类Unix系统里root居然会提示权限不足，真的不是在逗我玩嘛。

结果发现苹果坑爹的在新系统中加入了一个叫做rootless的机制。

<!--more-->

