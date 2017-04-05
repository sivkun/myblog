---
title: 'responsive web design 笔记 '
date: 2017-03-04 22:06:50
tags: [HTML,CSS, Markdown ]
categories: 技术
---
# 响应式网站设计

## 本文是在学习[responsive-web-design](https://github.com/sivkun/responsive-web-design)时，做的笔记。

## 响应式网站设计的优点
* 减少工作量
    网站、设计、代码、内容都只需要一份
    多出来的工作量只是js脚本、css样式做一些改动
* 节省时间
* 每个设备都能得到正确的设计
* 搜索优化

## 响应式网站设计的缺点
* 会加载更多的样式和脚本资源
* 设计比较难精准定位和控制
* 老版本浏览器兼容不好

## CSS3 媒体属性简介 略

## viewport视口

`布局视口（layout viewport）`: 虚拟的视口将页面呈现出来，布局视口的宽度是不变的。
`可视视口（visual viewport）`:  对布局视口呈现出来的页面进行放大或者缩小，在固定的设备显示器看到的网页的内容区域的内容范围或大或小。
 比方说：320px宽的显示屏，网页宽度为960px，在显示屏中显示这个网页，渲染时会按照网页960px的宽度进行渲染，然后将整个页面缩放三分之一，
 这样整个页面显示出来，此时可视视口为960px因为整个页面你都看到了，即布局视口宽度等于可视视口宽度。将页面放大，在320px的设备中，
 只会看到页面的一部分，可视视口变小。可视视口尺寸不固定，随着用户缩放页面改变。 （默认宽度是屏幕的宽度）
 像是拿着放大镜看书，书是布局视口，通过放大镜看到的是可视视口，随着放大镜离书的远近，可视视口改变，左右移动放大镜，可看到整个书的内容。
 这样的话用户浏览网页时，需要不断的滑动、放大页面，很不方便。
`理想视口（ideal viewport）`：布局视口在一个设备上的最佳尺寸。理想视口下的页面便于浏览器浏览阅读。
  理想视口就是为了构架手机浏览器优化的页面而添加的
<!-- more -->
```html
<meta name="viewport" content="width=device-width"/> <!--告诉设备要使用理想视口，该宽度等于屏幕的宽度。-->
```
如果不指定上面属性，布局视口的宽度是设备厂商的默认值，指定该属性布局视口成为理想视口。

```html
<meta name="viewport" content="width=device-width,minimum-scale=1.0,maximum=1.0,user-scaleable=no"/>
<!--最小缩放比例、最大缩放比例、 禁止缩放-->
```

渐进增强
优雅降级（推荐）

## 响应式网站设计实践原则
* 断点选择
例如：
  0~480 小屏幕
  481~800 中屏幕
  801~1400 大屏幕
  1400+ 巨屏幕

## 如何组织项目目录结构
   * 约定优于配置（convention over configuration）
    约定代码结构或命名规范减少配置数量
    css vs style
    img vs image 
    image vs images
    
## markdown语法 
dillinger.io 在线md编辑器
```html
> 引用名言

[百度](http://www.baidu.com)

![图片](src/favicon.icon)

**粗体** *斜体* ***粗体加斜体***

表格

|col1|col2|col3|
|---:|:---:|:---|
|col11111|col2dddddd|col333333|
|col1|col2|col3|
```
效果如下：

---

> 引用名言

[百度](http://www.baidu.com)

![图片](src/favicon.icon)

**粗体** *斜体* ***粗体加斜体***

表格

|col1|col2|col3|
|---:|:---:|:---|
|col11111|col2dddddd|col333333|
|col1|col2|col3|
---
## html5标签 
`<article>`标签是`<section>`标签的子集。
```html
<b></b>
<em></em>
<i></i>
```

* 在网页中，一些必不可少的图片使用`<img>`引入，可有可无的装饰性图片可以用标签的style引入。

## px em rem
* px 1个px相当于1个像素
* em 相对长度单位
 1. em相对参照物为父元素的font-size
 1. em 具有继承的特点
 1. 当没有设置font-size时，浏览器会有一个默认的em设置：1em=16px
 * em 的缺点：容易混乱
* rem
  1. rem的相对参照物为根元素html，相对于参照固定不变，所以较好计算
  1. 当没有设置font-size时，浏览器会有一个默认的rem设置：1rem=16px,这点与em是一致的
  font-size:62.5% 1rem=10px (10/16*100%)
  font-size:100% 1rem = 16px 
  

 ## 清除浮动,预防高度塌陷
1. 加标签： `<div style="clear:both"></div>`
1. 浮动容器加 `overflow:auto`
1. 包裹容器加 `float`
1. 包裹容器加after伪元素设置css`clear:both`

## BFC:  
相关内容一大堆 
> http://www.cnblogs.com/lhb25/p/inside-block-formatting-ontext.html
> http://www.zhangxinxu.com/wordpress/2015/02/css-deep-understand-flow-bfc-column-two-auto-layout/
> http://web.jobbole.com/84808/

在解释 BFC 是什么之前，需要先介绍 Box、Formatting Context的概念。  
* Box: CSS布局的基本单位  
　　Box 是 CSS 布局的对象和基本单位， 直观点来说，就是一个页面是由很多个 Box 组成的。元素的类型和 display 属性，决定了这个 Box 的类型。 不同类型的 Box， 会参与不同的 Formatting Context（一个决定如何渲染文档的容器），因此Box内的元素会以不同的方式渲染。让我们看看有哪些盒子：  
block-level box:display 属性为 block, list-item, table 的元素，会生成 block-level box。并且参与 block fomatting context；
inline-level box:display 属性为 inline, inline-block, inline-table 的元素，会生成 inline-level box。并且参与 inline formatting context；
run-in box: css3 中才有， 这儿先不讲了。
* Formatting context  
　　Formatting context 是 W3C CSS2.1 规范中的一个概念。它是页面中的一块渲染区域，并且有一套渲染规则，它决定了其子元素将如何定位，以及和其他元素的关系和相互作用。最常见的 Formatting context 有 Block fomatting context (简称BFC)和 Inline formatting context (简称IFC)。
* BFC(Block formatting context)直译为"块级格式化上下文"。  
它是一个独立的渲染区域，只有Block-level box参与， 它规定了内部的Block-level Box如何布局，并且与这个区域外部毫不相干。

`BFC有三个特性`
* BFC会阻止垂直外边距（margin-top、margin-bottom）折叠

    按照BFC的定义，只有同属于一个BFC时，两个元素才有可能发生垂直Margin的重叠，这个包括相邻元素，嵌套元素，只要他们之间没有阻挡(例如边框，非空内容，padding等)就会发生margin重叠。
因此要解决margin重叠问题，只要让它们不在同一个BFC就行了，但是对于两个相邻元素来说，意义不大，没有必要给它们加个外壳，但是对于嵌套元素来说就很有必要了，只要把父元素设为BFC就可以了。这样子元素的margin就不会和父元素的margin发生重叠了。

* BFC不会重叠浮动元素
* BFC可以包含浮动

我们可以利用BFC的第三条特性来“清浮动”，这里其实说清浮动已经不再合适，应该说包含浮动。
也就是说只要父容器形成BFC就可以，简单看看如何形成BFC

1. float为 left|right
1. overflow为 hidden|auto|scroll
1. display为 table-cell|table-caption|inline-block
1. position为 absolute|fixed
1. 我们可以对父容器添加这些属性来形成BFC达到“清浮动”效果

flex布局


##  导航栏的问题每一个标签之间出现空隙
```html
 <ul>
    <li><a href="#">登录</a></li>
    <li><a href="#">快速登录</a></li>
    <li><a href="#">关于</a></li>
    <li><a href="#">帮助</a></li>
    <li><a href="#">APP下载</a></li>
</ul>
 ```
 解决方法：
  1. `<li>`标签不换行
  1. `<li>`标签不闭合
  1. 设置`<ul>`字体font-size：0,内部`<li>`设置font-size；
  1. 设置`li+li{margin-left: -3px;}`负边距。
  1. css4属性中有white-space-collapsing
```css
 .notice a:first-child:before {
    content: '最新公告：\00a0\00a0'; /*\00a0空白字符*/
    color: #aaa;
}
```
`伪元素`  
与伪类针对特殊状态的元素不同的是，伪元素是对元素中的特定内容进行操作，它所操作的层次比伪类更深了一层，也因此它的动态性比伪类要低得多。实际上，设计伪元素的目的就是去选取诸如元素内容第一个字（母）、第一行，选取某些内容前面或后面这种普通的选择器无法完成的工作。它控制的内容实际上和元素是相同的，但是它本身只是基于元素的抽象，并不存在于文档中，所以叫伪元素。

 `伪类`  
伪类选择元素基于的是当前元素处于的状态，或者说元素当前所具有的特性，而不是元素的id、class、属性等静态的标志。由于状态是动态变化的，所以一个元素达到一个特定状态时，它可能得到一个伪类的样式；当状态改变时，它又会失去这个样式。由此可以看出，它的功能和class有些类似，但它是基于文档之外的抽象，所以叫伪类。

webp
svg
png
jpg
`响应式图片`方式
js或服务器
srcset
srcset配合sizes
picture
svg  【illustrator】

picturefill库

npm
^1.7.4 (大中小)  大   相同
~1.7.4          大中 相同  


`处理兼容性`

`同步测试工具`browser-sync
`browser-sync start --server "src"  --file "src"`

自动化构建工具 1、Grunt 2、gulp
静态资源打包工具 1、webpack

原型设计
axure
sketch
