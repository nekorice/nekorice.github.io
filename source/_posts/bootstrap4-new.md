---
title: bootstrap4的新增内容
date: 2018-02-14 20:44:10
tags:
- css
- 前端
---

不知不觉，boostrap已经走过了第5个年头。最近bootstrap4已经正式发布了。

这次bootstrap的大版本带来了哪些变化呢？是否要从bootstrap3迁移到4呢。

本着这些问题我去查阅了官方文档和一些相关站点。


<!-- more -->

总体来说，bootstrap4目前主要的更新有：

1. mobile first
grid使用rem和flex布局，主要面向移动端不同分辨率适配，并且响应式变成了5档。这里使用的是浏览器默认字体大小，也就是1rem=16px。flex布局的使用也让对齐和某些特殊布局更加方便。

2. 精简了结构
不再打包使用Glyphicons，使用图标需要自己引入第三方库，官方推荐[iconic](https://useiconic.com/open/)和[Octicons](https://octicons.github.com/)

3. 浏览器兼容性方面，官方支持到IE10+，文档说在IE9下也不会有异常表现，但是官方没有在这方面维护的计划，如果有显示bug可能需要自己去修复。

4. 整理了很多utils的class来支持更方便的样式显示和布局。比如flex，display，等等一些新的布局class。
也有一些常用的style属性的封装，比如border，position，float等等。

5. 支持响应式的float，size，spacing，每种实例class都支持sm,md,lg这种breakpoint。

6. 用sass替换了less,重载css改名为reboot，各种以前样式的移动端优化，和一些扩展。

整体来说不像2到3那样，破坏性的变化那么多。对使用者来说，具体实现细节从px到rem的变化，体验也不会太明显。

不过如果你的应用高度依赖于bootstrap的响应式宽度，可能就需要改动很多部分了。

下面我们详细看看grid和实用类的一些改动。

### Grid系统的修改

1) 响应式的宽度变成了5档。部分宽度col-lg有变化，col-xs移除了，改成了col-

** boostrap3 ** 

{% raw %}
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css" integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">

<table class="table table-bordered table-striped">
    <thead>
        <tr>
            <th></th>
            <th> Extra small devices <small>Phones (&lt;768px)</small> </th>
            <th> Small devices <small>Tablets (≥768px)</small> </th>
            <th> Medium devices <small>Desktops (≥992px)</small> </th>
            <th> Large devices <small>Desktops (≥1200px)</small> </th>
        </tr>
    </thead>
    <tbody>
        <tr>
            <th class="text-nowrap" scope="row">Grid behavior</th>
            <td>Horizontal at all times</td>
            <td colspan="3">Collapsed to start, horizontal above breakpoints</td>
        </tr>
        <tr>
            <th class="text-nowrap" scope="row">Container width</th>
            <td>None (auto)</td>
            <td>750px</td>
            <td>970px</td>
            <td>1170px</td>
        </tr>
        <tr>
            <th class="text-nowrap" scope="row">Class prefix</th>
            <td><code>.col-xs-</code></td>
            <td><code>.col-sm-</code></td>
            <td><code>.col-md-</code></td>
            <td><code>.col-lg-</code></td>
        </tr>
        <tr>
            <th class="text-nowrap" scope="row"># of columns</th>
            <td colspan="4">12</td>
        </tr>
    </tbody>
</table>
<br>
{% endraw %}

** bootstrap4 **

{% raw %}
<table class="table table-bordered table-striped">
  <thead>
    <tr>
      <th></th>
      <th class="text-center">
        Extra small<br>
        <small>&lt;576px</small>
      </th>
      <th class="text-center">
        Small<br>
        <small>≥576px</small>
      </th>
      <th class="text-center">
        Medium<br>
        <small>≥768px</small>
      </th>
      <th class="text-center">
        Large<br>
        <small>≥992px</small>
      </th>
      <th class="text-center">
        Extra large<br>
        <small>≥1200px</small>
      </th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <th class="text-nowrap" scope="row">Max container width</th>
      <td>None (auto)</td>
      <td>540px</td>
      <td>720px</td>
      <td>960px</td>
      <td>1140px</td>
    </tr>
    <tr>
      <th class="text-nowrap" scope="row">Class prefix</th>
      <td><code>.col-</code></td>
      <td><code>.col-sm-</code></td>
      <td><code>.col-md-</code></td>
      <td><code>.col-lg-</code></td>
      <td><code>.col-xl-</code></td>
    </tr>
    <tr>
      <th class="text-nowrap" scope="row"># of columns</th>
      <td colspan="5">12</td>
    </tr>
  </tbody>
</table>  
<br>
<br>
{% endraw %}


2）依然使用12 column。但是也可以不使用数字的class，将会根据col的个数自动等分。

{% raw %}
<div class="container">
  <div class="row">
    <div class="col-sm border">
      One of three columns
    </div>
    <div class="col-sm border">
      One of three columns
    </div>
    <div class="col-sm border">
      One of three columns
    </div>
  </div>
</div>
<br> 
{% endraw %}

```html
<div class="container">
  <div class="row">
    <div class="col-sm">
      One of three columns
    </div>
    <div class="col-sm">
      One of three columns
    </div>
    <div class="col-sm">
      One of three columns
    </div>
  </div>
</div>
```



3）如果不需要使用默认的row的gutters, 可以使用.no-gutters来去掉padding。

4) 因为使用了flexbox，row也可以使用align-items，justify-content来设定，这部分在下一个部分utilityies里的flex中再详细介绍。另外增加了一个用于在flexbox中强制换行的类“.w-100”

{% raw %}
<div class="container">
  <div class="row">
    <div class="col border">Column</div>
    <div class="col border">Column</div>
    <div class="w-100"></div>
    <div class="col border">Column</div>
    <div class="col border">Column</div>
  </div>
</div>
<br>
{% endraw %}

```html
<div class="container">
  <div class="row">
    <div class="col">Column</div>
    <div class="col">Column</div>
    <div class="w-100"></div>
    <div class="col">Column</div>
    <div class="col">Column</div>
  </div>
</div>
```


### Utilityies Class

## flex

增加了一些用于flexbox的类。

1） .d-flex, .d-inline-flex 设定显示为flexbox

2） 提供了flexbox属性的class,使用对应属性名加上值组成的class。
    比如 .justify-content-start

    justify-content： start|end|center|between
    align-items：
    align-self： start|end

关于flexbox布局，推荐css-tricks的一篇文章: [https://css-tricks.com/snippets/css/a-guide-to-flexbox/](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)

另外这些class都支持响应式布局的breakpoint class，即sm，md，lg那些。

3） 支持auto-margin。使用mr-auto（margin-right-auto），mb-auto，ml-auto等,可以按照内容的宽度，自动计算最大化的margin。

4） 还有warp和order等属性class

可以说bootsrap4把这些常用属性都class化了。这方面具体是否使用这些class来辅助布局，就仁者见仁智者见智了。

## spacing

看了一下文档也是把css属性，class化了。格式为：

    {property}{sides}-{breakpoint}-{size}

property中， m对应margin， p对应padding。

sides中, t,b,l,r对应top，bottom，left，right。另外还有x，y则是表示，同时设置left和right，或者top和bottom。

size, 0-5 则对应于几倍的基本边距。不过具体数字对应还比较神奇。

这里直接引用官方文档了。

    0 - for classes that eliminate the margin or padding by setting it to 0
    1 - for classes that set the margin or padding to $spacer * .25
    2 - for classes that set the margin or padding to $spacer * .5
    3 - for classes that set the margin or padding to $spacer
    4 - for classes that set the margin or padding to $spacer * 1.5
    5 - for classes that set the margin or padding to $spacer * 3
    auto - for classes that set the margin to auto

## display，float

基本上也是同样的概念，增加了一些对应css属性的class

    .d-{breakpoint}-{value}

比如display:inline, 对应.d-inline.

而float则对应于

    .float-{breakpoint}-{value}

## color

多了 secondary，light, muted几种颜色。


<div class="p-3 mb-2 bg-primary text-white">.bg-primary</div>
<div class="p-3 mb-2 bg-secondary text-white">.bg-secondary</div>
<div class="p-3 mb-2 bg-success text-white">.bg-success</div>
<div class="p-3 mb-2 bg-danger text-white">.bg-danger</div>
<div class="p-3 mb-2 bg-warning text-dark">.bg-warning</div>
<div class="p-3 mb-2 bg-info text-white">.bg-info</div>
<div class="p-3 mb-2 bg-light text-dark">.bg-light</div>
<div class="p-3 mb-2 bg-dark text-white">.bg-dark</div>
<div class="p-3 mb-2 bg-white text-dark">.bg-white</div>



这里只列出相对常用的内容，其他内容可以去查阅官方文档。

上述这些Utilityies Class，应当酌情使用。否则可能会导致html的class过于冗杂。

### 小结

bootstrap4还优化了form的表单显示。之前的form-control，form-group都有了一些小扩展。

总体来说bootstrap改动不大，只是官方文档使用了大量的Utilityies Class，初看起来可能有点混乱，实际了解之后

buttons，navbar控件都有一些小优化，这些就需要诸君在使用中再去查看细节了。





