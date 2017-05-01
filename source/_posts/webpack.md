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


## entry

作为入口定义的配置，webpack支持多个入口，可以根据需要打成几个不同的bundle，来按需加载。

比起整个站点都打成一个文件更加灵活。

todo

chunkfile的意义


## output

todo

filename 设置成了chunkhash之后如何映射到htmlfile里，--》 使用插件

libiry

libiryTarget

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

* url-loader

* html-loader

* bable-loader

等等

## externals

第三方依赖，希望不使用webpack来bundle的文件

可以使用resolve里的别名

todo
externals: ["react", /^@angular\//],
externals: "react", //

```
externals : {
  react: 'react'
}

// or

externals : {
  lodash : {
    commonjs: "lodash",
    amd: "lodash",
    root: "_" // indicates global variable
  }
}
```


##其他配置

###target

主要表示打出的文件的环境，默认就是web。除此之外还有node和electron-main等等。

有时候选择web会提示找不到fs和net模块的错误：

```
Cannot resolve module 'fs'
Cannot resolve module 'net'

``` 

这个时候在webpack的根配置项中加入如下的配置，就可以避免这个问题

```
target: "web",
node:{
    fs:'empty',
    net:'empty',
},
```

虽然把target改成node也可以解决这个问题，但是打出的bundle就不能够在web中使用了

具体参考：

###devServer

代理的设置和browsersync的配置基本是一样的，都是使用[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)进行配置，如果之前gulp也是使用browsersync来配置后端的反向代理，那么基本复制对应的配置段到proxy里就可以了。

静态文件的url path缺省则是由output中的publicPath设定来识别，也可以在devServer设置一个publicPath来改写这个配置
webpack打出的文件默认会在这个url path下面来serve，不用额外自己创建目录结构。

### 导入env变量

传入环境变量，来控制是否是生产环境还是开发环境

```
-module.exports = {
+module.exports = function(env) {
+  return {
    plugins: [
      new webpack.optimize.UglifyJsPlugin({
+        compress: env.production // compress only in production build
      })
    ]
+  };
};
```

---

最后贴一下自己用的代码

<script async src="//jsfiddle.net/nekorice/sd0535ux/embed/js/"></script>

