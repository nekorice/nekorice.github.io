title: js阻止页面刷新
date: 2017-11-15 23:08
tags:
- js

---
网页中用户的表单填写到一半，或者ajax请求发送期间，如果用户刷新浏览器可能会导致数据保存失败。需要阻止页面刷新，这时可以通过监听页面window.onbeforeunload事件函数来处理。

window.onbeforeunload = function(e) {
  var dialogText = 'Dialog text here';
  e.returnValue = dialogText;
  return dialogText;
};

不过chrome不支持自定义显示文字，固定为
“要重新加载该网站吗？
系统可能不会保存您所做的修改”
而firefox和safari就会正常使用return value里面的文字

参考:
-- [https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onbeforeunload#Browser_compatibility](https://developer.mozilla.org/en-US/docs/Web/API/WindowEventHandlers/onbeforeunload#Browser_compatibility)