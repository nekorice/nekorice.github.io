title: angularjs概览
date: 2017-03-06 23:34:39
tags:
- angularjs
- 前端
---

用了 angularjs(1.x) 也有一段时间了,然而之前的学习笔记计划全都荒废了.
趁着自己要对组内新人进行一些分享,记忆中的一些细节还没有变的模糊,赶紧总结一下个人的学习经验.

<!--more-->

#angularjs 基本概念

* **双向绑定**

双向绑定现在已经不是一个新概念,大部分时间我们不用考虑怎么去刷新 DOM, (妈妈再也不用担心我乱绑定事件而导致界面卡了)
而主要去关心模型的变化,还有如何写 HTML 的表现.
不过有时候里面混着一堆 angularjs 的语句看起来也挺蛋疼的.但相比后端套模板,你连输入的数据是什么都不知道还是要好一点的

当然,当把刷新权交给了框架,还是要了解一下什么时候 angularjs 会刷新我们的界面,而什么时候不会刷新,需要我们手动触发刷新($scope.$apply),

可以参考一下这篇文章 [AngularJS的scope.$apply](http://blog.gejiawen.com/2014/07/14/usage-for-angularjs-scope-apply/)

* **依赖注入**

依赖注入则更像是用一些奇技淫巧实现的js语法糖,来解决加载顺序的问题.以后有了 ES6 的import, 大概也就不需要了.

* module

module应该算是Angularjs最小逻辑结构.其他的组件和功能都是绑定在 module 上来实现的.

一般来说一个页面只有一个 ng-app 作为该块级元素的主 module, 当然你也可以定义多个.可以参考 [multiple-ng-app-within-a-page](http://stackoverflow.com/questions/18571301/angularjs-multiple-ng-app-within-a-page).

根据功能模块来划分不同 module, 绑定不同的 controller,directive,provide, 最后通过依赖注入,引用到主 module. 

当你习惯了 angularjs 的编程思路之后, 自然而然就实现了前端功能逻辑的解耦.

[angularjs学习笔记 -- module]()

* controller

喜闻乐见的 mvc 中的我们最熟悉的老朋友,恩,对它就是封装各种对模型操作和各种交互的地方.

也可能是我们一开始写代码时,容易写的最臃肿的地方.

不过还有什么地方能够比 controller 更像垃圾堆的地方了嘛╮(╯▽╰)╭

[angularjs学习笔记 -- controller]()

* directive

directive 本身也可以封装一些独立的逻辑,作为一个逻辑组件.

不过个人感觉它主要干的事情,是用来帮助我们解决一些重复的渲染,可以把它想象成一个 UI component 的封装.当然它能够做到的东西更多.

另外我们常用的 ng-repeat,ng-if,ng-class 等等这些东西,都是angularjs 内建的 directive.

[angularjs学习笔记 -- directive]()

* provider(Service,Factory,Constant,Value等等)

这块之前一直都是分开看的,一直到最近读了一篇文章才发现原来后面那些东西都是 provide 的一个特例.

主要就是用于在 controller 之间共享一些对象,数据或者服务.

因为是单例模型,所以可以保证数据的一致性.

provider的概念其实很绕,首先用 provider 方法定义的 provider, 可以用 config 方法,传入一个providerName 的 Provider 后缀的变量进行定制.

是不是听的很头晕.

可以参考官方的 provider, $http 和 $httpProvider. 官方文档里的 provider 指的是这些在 config 里面依赖注入的 Provider 变量,然而这其实只是 provider 的副产品.

[angularjs学习笔记 -- provider]()

* test

分了这么多模块,功能清晰了,然而如果出了问题,那么多的文件,你要去哪里 debug 呢?

所以模块化和单元测试是分不开的两兄弟.

不写单元测试的程序员,不是一个好美术~~

angularjs 的测试模块也是非常强大,不仅有模块的 BDD 单元测试, 还直接集成了Browser 的 E2E 测试.

[angularjs学习笔记 -- Jusmine和 单元测试]()
























