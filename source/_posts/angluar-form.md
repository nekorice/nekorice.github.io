title: angularjs的ng-model和表单验证(form-validate)
date: 2017-03-27 00:15:32
tags:
- angularjs
- 前端
- form
---

由于接触 angularjs, 并且拿它来制作的功能中很少有表单的应用.
所以有很长一段时间都没有怎么用到ng-modal, 只觉得这个directive就是个用来绑定 form 中的变量, 开发起来很方便.

然而今天开始更详细的使用 angularjs 来制作表单应用的时候,才发现自己原来把它想象的太简单了.

<!--more-->

## ngmodel

按照官方指南, ngmodel 主要的功能有以下几个部分:

* 绑定视图到 model 对象上, 需要依赖比如input, textarea 或者 select这些 directive.(原来这些标准 html 输入控件也被 angular 写成了一个 directive!)
* 提供表单验证(比如 html5标准的 required, number, email, url)
* 保存被绑定表单输入控件的状态(valid/invalid, dirty/pristine, touched/untouched, validation errors)
* 设置对应的 css class和动画(ng-valid, ng-invalid, ng-dirty, ng-pristine, ng-touched, ng-untouched, ng-empty, ng-not-empty)
* **注册**输入控件到它的父表单

原来之前我一直了解到的只有它的第一点,它还有这么多功能.

然而就连第一点,我都没有透彻的理解,实在是惭愧.

先从第一点讲起.其他几点则在第二部分表单验证里面说明.

ngmodel 可以在以下几种输入控件(已经被angular重写成了directive)中使用.

让我们过一遍这些已经耳熟能详的标准输入控件在 angularjs 里面被进行了什么样子的神奇改造.

### input

**[type=text]**

text 控件是其他 input 类型的基础,其他 input 控件大多数的属性是从这个属性的控件继承(directive的继承?)过去的.

```
<input type="text"
       ng-model="string"
       [name="string"]
       [required="string"]
       [ng-required="string"]
       [ng-minlength="number"]
       [ng-maxlength="number"]
       [pattern="string"]
       [ng-pattern="string"]
       [ng-change="string"]
       [ng-trim="boolean"]>

```

这里的 ng-model 只是 input 的一个参数.用于表明这个数据绑定到当前scope哪个变量中.

name, 很容易漏掉的一个属性,这个属性不只是原来的 input 的 name, 同时 angularjs 也会使用这个 name 注册到当前的 form(directive) 之中.

所以看起来一样的 input 和 form 以及 select 等等这些标准 html 控件实际上都被用 directive 包装了一层,而让所有的控件数据内容在 angularjs 的管理下.

仔细想想也是需要这样,才能方便的实现双绑.

required, ng-minlength, pattern 用来做表单验证的属性,后面统一讲.

ng-change 就是 onchange.

ng-trim input 自带了去掉首尾空格的功能,很好,很强大.也可以设置属性为 false 来关闭.

ng 属性和不带 ng 属性主要是否支持 angularjs 表达式的区别.

**[type==checkbox]**

```
<input type="checkbox"
       ng-model="string"
       [name="string"]
       [ng-true-value="expression"]
       [ng-false-value="expression"]
       [ng-change="string"]>
```

checkbox 就有点特别了

当没有设置 ng-true-value 和 ng-true-value, 选中 checkbox 会把绑定的变量设置成 true, 反之则为 false.

然而它居然没有默认的 value 属性, 也就是你设置 value 是不起作用的.只有设置ng-true-value 和 ng-true-value才能设置对应的值.

曾经就是在这里被坑到了...

另外如果设置对应 ng-model 绑定的变量属性为 true 或者 ng-true-value 均不能在重绘时使 checkbox 被设置为勾选??

这个时候就需要使用 ng-check 来设置初始值.

**[type==radio]**

```
<input type="radio"
       ng-model="string"
       value="string"
       [name="string"]
       [ng-change="string"]
       ng-value="string">
```

radio就比较中规中矩了,选择时,设置为 value 或者 ng-value.

**input[range]**

```
<input type="range"
       ng-model="string"
       [name="string"]
       [min="string"]
       [max="string"]
       [step="string"]
       [ng-change="string"]
       [ng-checked="expression"]>
```

标准的 rage 虽然用的不多,不过也有一些不同的地方

对于不支持 rage 控件的浏览器, rage 会变成一个普通的 input 框,依然包含 binding,validation 和数值的自动转换.
对于支持 rage 控件的浏览器,浏览器并不允许 rage 赋值成一个非法值.

也就是说,

1.任何非数值都会被赋值成 (max + min) / 2
2.任何数字超过 max 和 min 都会赋值成 max 或者 min.如果有 step 设置则是按照 step 能到的值来进行赋值.

所以这个控件永远不会有 requied 或者 min 和 max 的 error.

而且 range 也不能够使用 ngMax,ngMin,ngStep 这些 directive,因为这些 directive 并没有设置对应的属性,导致初始化 dom 时,浏览器会使用默认值min = 0, max = 100, and step = 1,来进行初始化.

基本上标准的 html5 的 rage 浏览器支持并不是很完善,推荐使用 css 和 js 来实现自定义的 rage 插件会更好一点.


### select

select 基本没有太多要讲的. 

ngModel 会绑定对应被选择 option 的 value.

一个问题就是使用 ng-repeat 还是 ng-options 来生成 options.

官方推荐是使用 ng-options. 因为 ng-options 使用了 DocumentFragment 来生成每个 options,可以减少内存占用和渲染占用时间,并且不会生成一个内置的循环用的 scope.

另外就是发现之前自己一直没有认真看 html, 原来 select 还有 size 这么一个属性,用于控制下拉列表的显示选项个数,超过这个个数就会出现滚动条,默认值是0,由浏览器自行控制.

其他控件大同小异,可以参考官方文档,这里就不多说啦.

---

## 表单验证

一个简简单单的 ngmodel, 就几乎做完了你所想到的表单验证几乎需要的全部功能.只要简单封装几乎就能够满足大部分需求.

简直不敢让人相信, angularjs 原来内置了一个这么有用的表单验证功能.

```
<form
  [name="string"]>
...
</form>
```

实际上 form 也被实现成了一个 directive, 所以在需要进行表单验证的 form 控件,设置一个 name

这样在当前 scope 里面的时候就会多出一个以 name 属性为命名的变量, 这个变量会存储当前 form 的验证结果.

另外虽然 html 里不允许 form 嵌套,但是在 angularjs 里可以使用 ngForm 标签来进行嵌套. ngForm 是 form directive 的别名.

其中 form 还会被设置一些 class

这里直接引用官网

```
ng-valid is set if the form is valid.
ng-invalid is set if the form is invalid.
ng-pending is set if the form is pending.
ng-pristine is set if the form is pristine.
ng-dirty is set if the form is dirty.
ng-submitted is set if the form was submitted
```

使用 ngSubmit 来进行提交



因为 model 绑定之后,任何输入都会导致 model 变化,有时候可能不想校验做的这么灵敏.只是希望在 blur 或者 click 事件之后才更新 model.
就要使用ngModelOptions这个设置, 来延迟 model 绑定的变化.


### 验证中可以使用的属性

只要控件都配置了各种验证条件,那么改 form 中就会有以下的属性来直接提供验证结果,把对应的变量用于绑定验证成功或者失败的提示以及 css, 即可几乎不用写 js实现表单验证.

$valid表示控件验证通过, $invalid则和$valid相反

$error 则为一个错误内容的字典,其中的key为对应的检测属性,值一般为 true, 不存在的错误不会在$error中.

比如 required, minlength, maxlength, pattern, min, max.

对于 input[type=email], 还有特殊的 email 类型的错误.
 

说了这么多,来看一个例子吧.


<iframe width="100%" height="600" scroll="no" src="//jsfiddle.net/nekorice/dq0f3s8L/embedded/" allowfullscreen="allowfullscreen" frameborder="0"></iframe>

















