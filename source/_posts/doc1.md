title: 第一篇博客
date: 2015-02-05 21:55:03
tags:
---



忙活了半天，终于搞定了。


[hexo](http://hexo.io/docs/) 真的是一个非常好用的静态博客系统。

评论到部署都非常方便，不需要多少编程基础就可以搞定。

<!--more-->

#关于 DISQUE 配置

1.注册 diqus 账号,然后点击右上角进入 admin 页面 [https://disqus.com/admin/]
2.在左边 Your Site 添加一个站点.
3.需要生成一个 disqus 的短地址,这里生成的就是你的 disqus shortname ,加入到你站点中 _config.yml
的 shortname 就可以自动生成 disqus 评论了.
4.如果你忘记了 shortname 可以在 site的 setting 中的 Basic 标签页中查看.

#关于 git 部署

其实可以不用官方插件,最后静态文件都生成在 public 目录里.
把 public 目录的内容都push到你的 github-page 资源库的根目录就可以了.
手写个脚本也可以一键部署

#关于自定义主题

_partial中的代码只是为了结构化,主界面主要是 layout 下面的7个页面,其中部分可以缺省.
每个页面文件对应一个链接.

最基础的是 index(主页) 和 post(文章)

js 和 css 则加在 layout 同级的 source 目录就可以.

目录的索引以 source 为模板的当前目录,相对路径直接从这里写起就可以

另外还是 swig 的模板阅读方便点, ejs 实在看得有点累

- - -

**参考** 

[更换博客系统——从jekyll到hexo](http://zhaofei.tk/2014/11/30/jekyll_to_hexo/)
[hexo你的博客](http://ibruce.info/2013/11/22/hexo-your-blog/)
[Hexo 3.0 靜態博客使用指南](http://ippotsuko.com/build-your-hexo-blog-3/)
