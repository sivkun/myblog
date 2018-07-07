---
title: transform
date: 2017-07-24 11:12:15
tags: [css3, transform,3d ]
categories: [css]
---
相关链接：
> http://www.cnblogs.com/xiaohuochai/p/5350254.html#anchor2
> http://www.cnblogs.com/xiaohuochai/p/5351477.html  
> 透视原理与实现 http://blog.csdn.net/goncely/article/details/5397729
> http://www.zhangxinxu.com/wordpress/2012/09/css3-3d-transform-perspective-animate-transition/

在实现carousel效果的时候，用到了transform:translate属性，哎，以前看过的都忘了，只有再重新拾起来，这里做个备忘。上面4个链接可以帮助理解。尤其是张鑫旭大神的，如果对透视不太理解，然后再结合透视原理那篇文章。
下面是用到的关键css属性：
```css
.parent{
  perspective:500px; /*视距，视角*/
  perspective-origin:center;/*视点*/
}
.child{
  transform-origin:50% 50%;/*变换中心*/
  /*transform-style 属性规定如何在 3D 空间中呈现被嵌套的元素。值：flat|preserve-3d,flat元素会被拍扁不会呈现3d效果，而preserve-3d可以呈现3d效果*/
  transform-style: preserve-3d;
  transform: rotateY(50deg) /*旋转，绕Y轴旋转50°*/
    scale(0.45)  /*缩放，为原来的0.45倍*/
    skewX(10deg) /*倾斜，在X轴倾斜，表示原来垂直X轴的变发生倾斜*/
    translate3d(10px,10px,-10px);/*平移，在空间中发生移动*/
}
```