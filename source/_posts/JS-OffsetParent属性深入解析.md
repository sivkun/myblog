---
title: JS OffsetParent属性深入解析
date: 2017-03-17 18:12:52
tags: [ js , css]
categorise: [技术]
---
[参考](http://www.jb51.net/article/45555.htm)

offsetParent属性返回一个对象的引用，这个对象是距离调用offsetParent的元素最近的（在包含层次中最靠近的），并且是已进行过CSS定位的容器元素。 如果这个容器元素未进行CSS定位, 则offsetParent属性的取值为根元素(在标准兼容模式下为html元素；在怪异呈现模式下为body元素)的引用。 当容器元素的style.display 被设置为 "none"时（译注：IE和Opera除外），offsetParent属性 返回 null。
语法：
parentObj = element.offsetParent
变量：
 parentObj 是一个元素的引用，当前元素的偏移量在其中计算。
主要结论：
* 当某个元素及其DOM结构层次中元素都未进行CSS定位时(absolute或者relative)[或者某个元素进行CSS定位时而DOM结构层次中元素都未进行CSS定位时],则这个元素的offsetParent属性的取值为根元素。更确切地说，这个元素的各种偏移量计算（offsetTop、offsetLeft等）的参照物为Body元素。(其实无论时标准兼容模式还是怪异模式，根元素都为Body元素)

* 当某个元素的父元素进行了CSS定位时（absolute或者relative），则这个元素的offsetParent属性的取值为其父元素。更确切地说，这个元素的各种偏移量计算（offsetTop、offsetLeft等）的参照物为其父元素

* 当某个元素及其父元素都未进行CSS定位时（absolute或者relative），则这个元素的offsetParent属性的取值为在DOM结构层次中距离其最近，并且已进行了CSS定位的元素。
