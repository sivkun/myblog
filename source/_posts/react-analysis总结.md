---
title: react-analysis总结
date: 2017-06-10 15:12:41
tags: [react,源码，框架]
categories: 源码分析
---
> ① http://blog.csdn.net/u013510838/article/details/55669742
> ② https://purplebamboo.github.io/2015/09/15/reactjs_source_analyze_part_one/
> ③ https://zhuanlan.zhihu.com/p/20346379?columnSlug=purerender
首先贴出我阅读源码时参考的链接。很感谢作者。
React源码规模十分庞大，开始阅读的时候不能直接陷进去，研究一些细节，而是要先把握整体的流程，对流程有基本了解后再沉下心仔细品味。这里推荐先去阅读上面的链接②，再去参考链接①。参考链接③讲了react  diff算法，可以参考react@0.14中对应的文件。

​	建议看React@15的源码。我一开始阅读源码的时候，把React源码clone了下来，版本比较高在16.0.0-alpha.3以后，源码中没有使用相对路径，用的所谓的全局模块系统，都找不到北。后来就找教程，参考如何阅读。后来就找到了上面两个，也并没有提什么全局模块系统。后来根据参考链接①，将react版本切换到了React@16.0.0-alpha.3。然后就看是研究源代码了。只看代码并不直观，需要配合调试，才能更好地理解。使用官方提供的`create-react-app`脚手架，进行编码测试。后来发现默认使用的是React@15.5.4，不过16里面也有对应代码，并且我已经添加了好多注释，并不影响阅读，也就没有再切换版本，当看到`ReactDOMComponent`为元素注册事件的时候，就看不下去了，乖乖的直接看测试项目下的`node_modules`中的`react`和`react-dom`模块去了。

​	React@16这个版本，其实发生了很大的变化，React将要发布fiber特性。React fiber对以前React的核心算法做了全新的改版。不过React@16.0.0-alpha.3代码库中，以前的模块还都存在，可以阅读研究。

<!--more-->

​	阅读React源码，首先对React源码中的基本概念有充分的理解。我想既然想研究React源码，肯定对已经对React有了充分的认识。如果有不明白的地方，[官方文档](https://facebook.github.io/react/docs/installation.html)是很好的资源。

# 关键方法

在代码中加debugger，console.log。在浏览器控制台调试。可以加在自己写的文件中，也可以加在`node_modules`中的`react`和`react-dom`模块源码中。

# React中的基本概念

1. `React Component`：用户自定义组件，以前是通过`React.createClass`创建自定义组件,现在es6语法是通过class 继承的语法创建自定义组件，自组件中可以定义生命周期函数等，自定义组件只是一个类或者说是一个构造函数，可以在JSX中声明式的使用，如`<div><MyConponent1/><MyComponent2/></div>`，这时并没有对自定义组件进行实例化。
2. `ReactElement`：`ReactElement`是React对用户自定义组件和原生DOM的一个抽象元素，就像原生DOM元素（div、span）一样可以构成DOM树，通过`ReactElement`之间的组合抽象形成`Virtual DOM`树。无论是DOM元素还是用户自定义组件都会被转化为ReactElement。

React以JSX形式书写，是要经过babel转义处理的。可以在http://babeljs.cn/repl/或者http://babeljs.io/这两个网站，进行观察测试。

上面只是概念，并不直观，可以自己测试，在控制台中将React中的重要的数据结构打印出来。`var result=ReactDOM.render(<App/>,...)`调用返回结果result是自定义组件App的实例化对象。在阅读源码的过程中，可以进行参考理解。可以看看React内部是如何表达`Virtual DOM`的树形结构的。

昨天在掘金看到的一篇文章[图解 React Virtual DOM](https://juejin.im/entry/592ea028a22b9d0057753349)对理解`Virtual DOM`会有些帮助。

# React源码中关键词汇

1. `reconsiler`:调度者或者调和着，reconciliation（n. 和解；调和；和谐；甘愿） algorithm：可以理解为调度算法

2. `publicInstance`：用户自定义组件实例化组件。
3. `internalInstance`：ReactElement对应的内部实例，也可以说是用户自定义组件对应的内部实例。内部实例是`ReactCompositeComponent`、`ReactDOMComponent`等实例化对象。

`publicInstance`和`internalInstance`这两个对象是相互引用的。

`App`实例就是`publicInstance`，`_reactInternalInstance`就是`App`引用的内部实例。

`_reactInternalInstance`有个`_instance`属性，是对`App`实例的引用。

4. `context`
  `contextType`:设置该属性的组件，则在该组件内部可以通过this.context获取到contex。
  `childContextTypes`:设置该属性的组件，可以设置向下传递context对象的属性的类型。
  `getChildContext()`：该函数设置可以向下传递的context对象属性。必须要在childContextTypes设置属性类型。
  特点：只要在一个组件设置了context，则在其后代组件中都可以获取到。
  ​       如果后代组件也设置了context，向下传递的context是`Object.assign({}, currentContext, childContext);`
  ​       `currentContext`:是从父组件传下来的Context（unmaskedContext）。
  ​       `childContext`:是当前组件设置的childContext。
   当前组件对应的`internalInstance._context=currentContent`
   当前组件`publicInstance`只能根据`contextType`，从`currentContext`生成对应的`context`。

5. `Transaction`：事务

  React中挂载、更新等都是以事务形式进行。可以理解为：一个操作如果以事务形式执行，那么可以在这个操作执行前进行一些初始化工作（initial阶段），在操作执行后进行一些擦屁股操作（close阶段）。具体实现看源码。

# React Mount阶段

大致流程：

1. 判断是否已经执行过ReactDOM.render()，如果是进行更新操作。否则执行3
2. 判断是否是服务器端渲染，如果是进行标记。执行3
3. 执行新组件的挂载操作。实例化组件，初始化批量挂载组件环境。 `ReactUpdates.batchedUpdates`,作用是：以事务形式进行，挂载操作，如果有setState类似更新组件操作，在close阶段进行事后处理。
4. 将调度挂载阶段事务化。  `ReactReconcileTransaction` 即 ReactReconcile事务：显然，是将ReactReconcile这一过程的事务化。作用：①维护一个callback对列，在挂载完成后在事务的close阶段调用。当然还有其它作用。
5. 递归挂载组件返回markup。 `ReactReconciler.mountComponent`顾名思义 React调和 挂载组件 ，调和的过程是React思想的精髓。diff过程融入其中。参见：[reconciliation](https://facebook.github.io/react/docs/reconciliation.html)
6. 并将markup插入container。这一步中，如果是服务器端渲染，进行校验和判断。
7. 执行 `ReactReconcileTransaction`的close阶段。
8. 执行 `ReactUpdates.batchedUpdates`的close阶段。

这是我开始研究源码时，在https://www.processon.com上画的图，
![ReactMount阶段分析过程图](http://www.processon.com/chart_image/591304bde4b0f320c44ff26f.png)
很是庞大，操作很复杂，不画图的话会跳晕的。当中可能有些不准确的地方。

还有实例化ReactComponent的过程，还有React组件的树形结构。如图


![InstantiateReactComponent](http://www.processon.com/chart_image/5915e122e4b0ef971ac9bb85.png)


# React updating阶段

可以结合一开始给出的参考链接①③阅读源码。还有总结里面给出的连接可以参考。

# React的合成事件

React实现了一套复杂的事件系统。可以结合一开始给出的参考链接①阅读源码。

# 总结

React源码很复杂，我也只是有了个大概的理解，还没能从更高的层次去理解其设计的原理及原因。

添加注释的代码参考https://github.com/sivkun/react-analysis。