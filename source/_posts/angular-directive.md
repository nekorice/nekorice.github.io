title: angularjs自定义directive
date: 2016-01-27 23:13:47
tags:
- angularjs
- directive
- 前端
---

最近要用 angularjs 制作一个报告模块,而表格插件还是使用基于 jquery 的插件.

页面使用了 angularjs 的路由来加载页面.虽然可以在 onload 函数中编写图表的加载,但是这样很不优雅,也不方便维护.如果直接用 directive 来绘制图表和其他数据,这样后台就可以直接考虑把表格数据渲染在 html 上.不仅可以减少一部分数据接口api.也方便做静态化.

<!--more-->

#directive 的定义

参考[官方文档](https://docs.angularjs.org/guide/directive)

定义一个 directive 的基本格式很简单,第一个参数是 directive 的 name, 第二个参数是一个函数,这个函数需要返回一个 object. 

```javascript
.directive('myCustomer', function() {
  return {
    templateUrl: 'my-customer.html'
  };
});
```

也可以写成依赖注入的数组形式.

```
.directive('myCustomer', [$timeout, function($timeout){
  return {
    templateUrl: 'my-customer.html'
  }
}])

```

directive 的 name 应当使用驼峰命名法,在 angularjs解析 html 时,以':','-','_'分割单词,和 data 或者 x 前缀都会被识别为对应 name 的 directive.

但是如果中间没有分割,比如写成 ngModel 则无法识别出来. 这样处理的原因,可能是因为 html 对大小写不敏感.

```
<div ng-controller="Controller">
  Hello <input ng-model='name'> <hr/>
  <span ng-bind="name"></span> <br/>
  <span ng:bind="name"></span> <br/>
  <span ng_bind="name"></span> <br/>
  <span data-ng-bind="name"></span> <br/>
  <span x-ng-bind="name"></span> <br/>
</div>
```


#directive 的参数配置

在第二个参数中函数返回的 object 里定义directive 的配置.

常见的参数有

```
restrict: 'AEC',
* E - Element name (default): <my-directive></my-directive>
* A - Attribute (default): <div my-directive="exp"></div>
* C - Class: <div class="my-directive: exp;"></div>
* M - Comment: <!-- directive: my-directive exp -->
```

是否替换掉原有的 directive 标签
```
replace: True/False
```

是否把标签的内部元素也传入 directive 进行处理
```
transclude: True/False
```

模板字符串/模板 url(只能使用其中一个参数)
```
template:""
template_url:""
```

绑定行为(link/compile二选一)

```
link: Object | function
compile: Object | function
```

完整的参数参考下述页面
[$compile 页面](https://docs.angularjs.org/api/ng/service/$compile)

##directive scope 

### 继承 scope

scope 可以取三种值, true, false 和一个 Object{}

当为 true 时,则继承 controller 的 scope, 通过访问 parent 来访问 controller 中的 scope, 所有修改都不会影响原有的 scope.

为 false 时,直接使用当前的父 controller 的 scope, 会共享之间的修改.

###isolated scope

directive 虽然可以访问模块的 scope, 官方建议通过属性把值传递到directive 的isolated scope中.

参考官方文档

```

.directive('myCustomer', function() {
  return {
    restrict: 'E',
    scope: {
      customerInfo: '=info'
    },
    templateUrl: 'my-customer-iso.html'
  };
});

<div ng-controller="Controller">
  <my-customer info="naomi"></my-customer>
  <my-customer info="igor"></my-customer>
</div>

```
显示为

```
Name: Naomi Address: 1600 Amphitheatre
Name: Igor Address: 123 Somewhere
```

isolated scope 一共有以下几种传入方式
``` 
'='  属性的值使用 angularjs 表达式计算
'@'  直接使用属性的字符串
'&'  则是可以传递函数

```

## link 函数

在 directive 生成模板的时候,需要一些额外处理,可以在 link 函数中处理.

需要注意的是, link 函数调用的时候(Post-linking function), 

这个时候如果获取 template_url 中的元素,

directive 是不一定完成渲染的,

如果这个时候去取子元素,很可能得到的会是空值.

要保证 post-link 里面可以获得子元素,应当使用 template 来作为模板的渲染.而不是一个 url.

#### 如何保证在 directive 渲染完成后执行函数

实在不能避免使用 template_url 来定义模板,目前一种可行办法是使用$timeout service

使用$timeout,这个 service, 当后面不带时间参数时,

定义的回调函数将在 angularjs 渲染完成之后调用.

相当于 jquery 的 document.ready()函数

```
.directive('myCustomer', [$timeout, function($timeout){
  return {
    templateUrl: 'my-customer.html',
    replace: true,
    link: function (scope, element, attrs) {
      $timeout(function (){
        //do something when angularjs loaded
      })
    }
  }
}])

```

## 实时更新 directive

默认 directive 只在生成的时候 link.

需要实时更新则要使用 scope.watch,来检测 scope 的修改

```

.directive('myCustomer', function(){
  return {
    templateUrl: 'my-customer.html',
    replace: true,
    link: function (scope, element, attrs) {

      scope.$watch(attrs.filter, function (newValue) {
        //filter属性可能是用其他的 scope 中传递过来的值        
        scope.filter = newValue
      })

    }
  }
})

```

## 如何动态获取 directive 的 name

比如定义了几个功能相似的 directive, 又不想用属性来区分,而是想要用 directive name 来区分.

我定义了一个 chart 的 directive, 但是我还想定义几个叫做 pie,bar 和 line 的 directive 
来作为 chart type='pie'|'bar'|'line' 的别名,这个时候想要动态获取一下 directive 来带入到一个生成 directive 的函数中.

需要动态获得 directive 的 name,link 函数无法获取 directive 的name, 这个时候就可以使用compile 函数.

```
compile: function(cElem, cAttrs, transclude) {
      //get directive name
      var name = this.name;
      //link func
      return function (scope, element, attrs) {
          
          //do somethin
          myFunc(name)

        });
      }
    }// end compile
```

compile 函数定义就是会返回一个 link 函数,而 compile 的this 指针指向的就是 directive 对象.利用闭包就可以传递 name.

需要注意的是当使用 compile 参数时, link 定义将会失效.















