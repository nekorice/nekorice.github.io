title: webpack2学习笔记
date: 2017-04-11 00:57:32
tags:
- webpack
- 前端工程化
---

之前一直在用 gulp 来进行前端持续集成和打包.

对于分散在各地的 html 模板没有什么好方法合并.

最近刚好有一个可以自由使用技术的新项目,打算用 webpack 来试试.

<!--more-->

之前也关注过 webpack 一段时间,没想到到自己实际使用的时候,已经是 webpack2.x 了.

google 搜索到的文档很多都是 webpack1.x 的,本来只是想写 webpack2的语法,结果还要去看 1.x 到2.x 的 migrate.

[https://webpack.js.org/guides/migrating/](https://webpack.js.org/guides/migrating/)

## resolve 配置

### module

查找模块的文件夹从modulesDirectories变成了module, 类似于 python 的 sys.path, require 模块的查找路径.

### alias

起个别名,来方便简写, 节省力气打字.

### extensions

数组,表示省略的后缀名,默认只能省略 .js,.json





## loaders(rule) 配置

webpack 里面改成了 rule, 但是仍然兼容 loaders 的写法.

毕竟这些插件都还是叫 loader, 每添加一种新的 loader, 都要记得去 npm install 对应的 loader.

rules 语法比起 loaders 语法, 也就是把级联和参数整理的更清晰了一点.

```diff
 module: {
-   loaders: [
+   rules: [
      {
        test: /\.css$/,
-       loaders: [
-         "style-loader",
-         "css-loader?modules=true"
+       use: [
+         {
+           loader: "style-loader"
+         },
+         {
+           loader: "css-loader",
+           options: {
+             modules: true
+           }
+         }
        ]
      },
      {
        test: /\.jsx$/,
        loader: "babel-loader", // Do not use "use" here
        options: {
          // ...
        }
      }
    ]
  }
```

pre-loader 和 post-loader 去掉了,变成了 rule的一个属性

常用的 loaders 有:

* style-loader, css-loader, less-loader, sass-loader

* 


##其他配置

###target

---

最后贴一下自己用的代码

<script async src="//jsfiddle.net/nekorice/sd0535ux/embed/js/"></script>

