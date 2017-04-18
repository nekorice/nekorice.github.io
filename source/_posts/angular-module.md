title: angularjs学习笔记 -- module
date: 2017-03-21 00:33:31
tags:
- angularjs
- 前端
---

module 是 angularjs 里面用来组织功能的一个集合.

可以把一打的功能(包含 controller,directive,service 等等),封装到一个 module 里,来对外部提供一个功能或者一系列功能.

我们所使用的大部分的  angularjs 的插件都是一个 module, 这样当我们把一个 module 依赖注入到我们的模块中,就可以使用这个 module 的全部功能.

<!--more-->

## 1.entry point

如同大部分程序都会有一个 main 函数.或者说像使用 browserfy 打包也需要一个 main.js. 

同样我们也需要一个 main module,作为我们 app 的入口.

这个 main module 是使用 ng-app 来定义的.

```javascript

<div ng-app="myApp">
  <div>
    {{ 'World' || greet }}
  </div>
</div>

```

这样 angularjs 就会从这个对应 ng-app 名称的 module 开始加载我们的模块.通过层层的依赖注入,初始化各个模块,各个 service, 然后生成虚拟 dom, 渲染我们的页面.

## 2.run 和 config

module除了常见的使用.链式操作来定义自己的 Service, Controller, Directive, Filter等等,主要就有额外的两个方法.

config 和 run

**run**

run 函数很容易理解,它就是 module 开始初始化时调用的函数,相当于 init 函数.在这里我们可以为我们的模块进行一些初始化的功能.需要注意的是,在 run 函数调用时所有的 provider (Service,Constant,Filter,Provider等等) 是已经初始化好了的.

所以可以直接使用以来注入,来使用 service 中提供的方法或者类.

比如官方的这个例子
```javascript

angular.module('xmpl.service', [])

  .value('greeter', {
    salutation: 'Hello',
    localize: function(localization) {
      this.salutation = localization.salutation;
    },
    greet: function(name) {
      return this.salutation + ' ' + name + '!';
    }
  })

  .value('user', {
    load: function(name) {
      this.name = name;
    }
  });

angular.module('xmpl.directive', []);

angular.module('xmpl.filter', []);

angular.module('xmpl', ['xmpl.service', 'xmpl.directive', 'xmpl.filter'])

  .run(function(greeter, user) {
    // This is effectively part of the main method initialization code
    greeter.localize({
      salutation: 'Bonjour'
    });
    user.load('World');
  })

  .controller('XmplController', function($scope, greeter, user) {
    $scope.greeting = greeter.greet(user.name);
  });
```

**config**

config 方法则调用的更早,是在各种 provide 初始化的时候,对 provider 进行配置.

也可以通过注入 $provide, $compileProvider, $filterProvider几个官方 provider 来注册 provider.

这句话很绕,这三个以来注入的官方功能也是一个 provider.用这个 provider 可以给模块直接注册更多的 provider.

比如官方文档这里的例子

```javascript

angular.module('myModule', []).
  value('a', 123).
  factory('a', function() { return 123; }).
  directive('directiveName', ...).
  filter('filterName', ...);

// is same as

angular.module('myModule', []).
  config(function($provide, $compileProvider, $filterProvider) {
    $provide.value('a', 123);
    $provide.factory('a', function() { return 123; });
    $compileProvider.directive('directiveName', ...);
    $filterProvider.register('filterName', ...);
  });
```

这两种方法是等价的.

angularjs中一个模块的加载顺序大致是这样

```
1. app.config()
2. provider.$get
3. app.run()
4. directive's compile functions (if they are found in the dom)
5. app.controller()
6. directive's link functions (again, if found)

```

config 更主要的作用就是 config provider.(是否感觉我又手抽了),因为 config 方法调用是在 provider 初始化($get)之前,所以可以在 config 方法中修改一些provider初始变量,来达到配置 provider 表现的目的.


provider 估计是 angularjs 里面最容易绕晕的概念,然而不用怕,下面这篇文章会带你完整揭开 provider 的面纱(大概).

[angularjs学习笔记 -- provider]()


> 附录

> [angularjs总览](/2017/03/06/angular-main/)



