title: Mac下使用sudo提示permission denied
date: 2017-04-21 10:06:21
tags:
- mac
- rootless

---

最近入手了一台新的macbook，升级了系统之后居然发现了一个诡异问题。

系统默认安装的six（python库）居然不能够升级，使用sudo（ All:（All，ALl））提示我权限不足。在类Unix系统里root权限居然会提示权限不足，真的不是在逗我玩嘛。

当然我可以选择使用virtuallenv来安装python的多环境配置，可是自己的计算机居然不在自己的掌控之下，感觉实在是不爽.

既然是拿来作开发机，总要研究清楚，不断的google中终于找到了解决方案。

结果发现苹果坑爹的在新系统中加入了一个rootless的机制。

<!--more-->

基本就是为了防止程序获取root权限对几个系统关键目录做出修改，被保护的目录主要是以下几个目录：

* /System
* /usr
* /bin
* /sbin
预安装的app（比如Appstore，iTunes等等）

如果你想要自己修改下面这些目录的内容，就需要关闭内核里面的rootless，也就是System Integrity Protection的服务。

方法如下：

1. 重启电脑，并且按下command+R，直到苹果logo出现。这个时候就会进入到Recoverty Mode。
2. 选择一个语言。
3. 进入恢复模式后，在上面的菜单找到实用程序（Utilities），在里面找到终端（Terminal）
4. 打开终端输入以下指令,关闭SIP
```
csrutil disable
```
5. 重启你的电脑，收工。

参考资料：

[https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-os-x-10-11](https://apple.stackexchange.com/questions/208478/how-do-i-disable-system-integrity-protection-sip-aka-rootless-on-os-x-10-11)

[https://support.apple.com/en-us/HT204899](https://support.apple.com/en-us/HT204899)

