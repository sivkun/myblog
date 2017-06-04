---
title: 节流器throttle
date: 2017-06-04 18:13:13
tags:
---

节流模式（Throttler）：对重复的业务逻辑进行节流控制，执行最后一次操作并取消其他操作，以提高性能。
用处：浏览器中的scroll、mouseover、mouseenter、mouseleave、resize等事件的处理函数被连续重复触发，很多情况下，只需要对最后一次触发做出反应，这时就要用到节流模式。
简单例子如下：
```html
<!DOCTYPE html>
<html lang="en">

<head>
  <title></title>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
</head>

<body>
  <script>
    function throttle(isClear, fn, param) {
      if (typeof isClear === 'boolean') {
        fn.__throttleID && clearTimeout(fn.__throttleID);
      } else {
        param = fn;
        fn = isClear;
        var p = Object.assign({}, {
          context: null,
          args: [],
          time: 300,
        }, param);
        throttle(true, fn);
        fn.__throttleID = setTimeout(function () {
          fn.apply(p.context, p.args);
        }, p.time);
      }
    }
    function moveHandler(e){
      console.log('move:',e.offsetX,e.offsetY);
    }
    window.onmousemove=function(e){
      throttle(moveHandler,{args:[e]});
    }
  </script>
</body>

</html>
```
思想：节流器的核心思想是创建计时器，延迟回调函数执行。