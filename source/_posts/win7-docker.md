title: 在windows7下使用docker
date: 2017-11-15 22:49:32
tags:
- docker
- windows7
---


#### 安装
---
最近学习redis，要在公司的windows工作机上面配个环境，刚好最近有在服务器上使用docker，看到docker也有windows版本，就想着直接用容器来弄个环境，应该比虚拟机配环境方便。

在win7下面使用的话，要使用 [docker tool](https://www.docker.com/products/docker-toolbox) 来安装。默认的product页面里的Docker CE 是在win10下面使用的。官方很不厚道的用灰色小字写在下面，害我查了半天的问题

quick start terminal需要使用bash.exe进行初始化，如果系统里没有安装git，安装时请勾选安装git。

之后的初始化需要翻墙去下载boot2docker.iso。

不能翻墙的同学可以去这里[https://github.com/boot2docker/boot2docker/releases](https://github.com/boot2docker/boot2docker/releases)手动去下载boot2docker.iso镜像，记得下载latest release版本，不要下载pre_release版本，否则启动检测依然会通不过。

<!-- more -->

然后把下载的iso放到C:\Users\你的用户名\\.docker\machine\cache\这个目录下。就可以通过启动检查了。

toolbox里面自带的kitematic挺有用的，可以启动关闭容器，快速修改环境变量和容器参数，十分方便。不过要注意kitematic默认exec执行的是sh，不是bash，要手动再输入以下bash，才有平时的操作。

#### 镜像和加速器
---

官方的docker hub也是被墙的不要不要的，推荐几个国内的hub。

https://c.163.com/hub#/m/home/       （需要登录）
https://hub.daocloud.io/

或者配置镜像加速器直接拉官方的docker hub镜像。

输入以下指令进入docker-machine
```bash
docker-machine ssh default
sudo vi /var/lib/boot2docker/profile
```
在EXTRA_ARGS里添加mirror配置，关于mirror可以参考这篇文章[https://ieevee.com/tech/2016/09/28/docker-mirror.html](https://ieevee.com/tech/2016/09/28/docker-mirror.html)
（这里使用官方的中国mirror）

```
EXTRA_ARGS='
--label provider=virtualbox
--registry-mirror=https://registry.docker-cn.com
'
```

保存文件，然后退出docker-machine，并重启
```bash
exit
docker-machine restart
```
重启完成之后，pull官网镜像就会快多了。可以开心的在windows下一键安装各种服务组件了~


