---
title: JS clientWidth offsetWidth offsetLeft clientX offsetX等差异
date: 2017-03-17 19:41:21
tags: [js,DOM]
categories: [技术]
---
![element](JS-clientWidth-offsetWidth-offsetLeft-clientX-offsetX等差异/element.jpg)

| 属性/方法                | 描述                                    |
| -------------------- | ------------------------------------- |
| element.clientHeight | 返回元素可见高度                              |
| element.clientWidth  | 返回元素的可见宽度                             |
| element.offsetHeight | 返回元素的高度                               |
| element.offsetWidth  | 返回元素宽度                                |
| element.offsetLeft   | 返回元素的水平偏移位置                           |
| element.offsetParent | 返回元素的偏移容器                             |
| element.offsetTop    | 返回元素的垂直偏移位置                           |
| element.scrollHeight | 返回元素的整体高度。                            |
| element.scrollLeft   | 返回元素左边缘与视图之间的距离。                      |
| element.scrollTop    | 返回元素上边缘与视图之间的距离。                      |
| element.scrollWidth  | 返回元素的整体宽度。                            |
| event.clientX        | 返回当事件被触发时，鼠标指针的水平坐标。（相对浏览器）           |
| event.clientY        | 返回当事件被触发时，鼠标指针的垂直坐标。（相对浏览器）           |
| event.pageX        | 返回当事件被触发时，鼠标指针的水平坐标。（相对文档，非标准但广泛支持，IE<9无）           |
| event.pageY        | 返回当事件被触发时，鼠标指针的垂直坐标。（相对文档，非标准但广泛支持，IE<9无）           |
| event.screenX        | 返回当某个事件被触发时，鼠标指针的水平坐标。(相对屏幕)          |
| event.screenY        | 返回当某个事件被触发时，鼠标指针的垂直坐标。（相对屏幕）          |
| event.offsetX        | 发生事件的地点在事件源元素的坐标系统中的 x 坐标(相对于鼠标点击的元素) |
| event.offsetY        | 发生事件的地点在事件源元素的坐标系统中的 y 坐标(相对于鼠标点击的元素) |
![event](JS-clientWidth-offsetWidth-offsetLeft-clientX-offsetX等差异/event.png)
