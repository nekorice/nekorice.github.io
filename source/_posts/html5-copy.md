---
title: html5复制内容到剪贴板
date: 2018-02-14 21:57:21
tags:
- 前端
- html5

---

很久之前web使用复制粘贴都需要一个隐藏的flash来提供功能，而html5则提供一个接口可以复制。

这个接口有一个库[clipborad.js](https://clipboardjs.com/)已经封装的很好了。不过这个库必须要把事件绑定到一个元素上，而不能够只提供一个复制的函数供我们调用。

对于现在的mvvm系统，实际事件都是绑定在虚拟dom上面了，所以这套基于dom事件响应的有时候用的不是特别顺手。

为了抽离出复制函数，需要搞清楚这其中的原理。

<!-- more -->

查看插件源码文件clipboard-action.js

主要核心复制函数就是

    document.execCommand()

查阅mdn，execCommand可以执行很多命令，不仅有copy，还有cut，paste，甚至bold之类。

而execCommand执行的命令，都是针对当前鼠标选择内容进行的。

那么要如何操作选择内容呢。

clipboard.js中是建立了一个隐藏的position:fixed,在屏幕外的元素保存复制内容，然后使用第三方库select来选择该元素的内容，然后执行copy复制的。

同时搜索mdn中的[selection](https://developer.mozilla.org/en-US/docs/Web/API/Selection)，也查到一些资料。

可以看到Selection是包含多个选择的Range。可以通过addRange，增加选择部分。

而select.js里面的代码也是直接使用了此api。

```javascript
//code1
element.select();
element.setSelectionRange(0, element.value.length);

//code2
var selection = window.getSelection();
var range = document.createRange();

range.selectNodeContents(element);
selection.removeAllRanges();
selection.addRange(range);

selectedText = selection.toString();
```


这样想要自己实现一个copy函数，只要创建一个隐藏元素，并且把它的内容设置成需要复制的值，再加入选择，之后执行execCommand，即可。


而浏览器兼容性，根据mdn，在IE9+，chrome 43+，firefox 41+，safari 10+上都是可以支持的。







