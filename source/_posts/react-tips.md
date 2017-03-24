---
title: react tips
date: 2017-03-04 22:40:18
tags: [ react ]
categories: 技术
---
#  `判断数据项是否是state `  

简单地对每一项数据提出三个问题：  
* 是否是从父级通过 props 传入的？如果是，可能不是 state 。
* 是否会随着时间改变？如果不是，可能不是 state 。
* 能根据组件中其它 state 数据或者 props 计算出来吗？如果是，就不是 state 。

#  `记住： React 中数据是沿着组件树从上到下单向流动的。`

可能不会立刻明白哪个组件应该拥有哪些 state 数据模型。
这对新手通常是最难理解和最具挑战的，因此跟随以下步骤来弄清楚这点：   
对于应用中的每一个 state 数据：
* 找出每一个基于那个 state 渲染界面的组件。
* 找出共同的祖先组件（某个单个的组件，在组件树中位于需要这个 state 的所有组件的上面）。
* 要么是共同的祖先组件，要么是另外一个在组件树中位于更高层级的组件应该拥有这个 state 。
* 如果找不出拥有这个 state 数据模型的合适的组件，创建一个新的组件来维护这个 state ，然后添加到组件树中，层级位于所有共同拥有者组件的上面。

# state更新

* 不要直接更改state的值
```javascript
// Wrong
this.state.comment = 'Hello';];
// Correct
this.setState({comment: 'Hello'});
//只有在构造函数中可以为this.state赋值
```
* state更新可能是异步的
```javascript
// Wrong
this.setState({
  counter: this.state.counter + this.props.increment,
});
// Correct 
//@param  prevState是state的前一个状态
//@param  props是当前传入的this.props的值
this.setState((prevState, props) => ({
  counter: prevState.counter + props.increment
}));
```

#  `React中函数式声明组件`
> https://segmentfault.com/a/1190000006180667

React中的每一个组件，都包含有一个属性（props），属性主要是从父组件传递给子组件，
在组件内部，我们可以通过this.props获取属性对象。

#  `React 中 context 的使用`

为了有时候想传递数据通过组件树，但是不想给每一层级的组件手动传递属性，
那么 context 就能帮你 "越级" 传递数据到组件树中你想传递到的深层次组件。
有时候 A组件 为了给 B组件 中的 C组件 传递一个 prop ，
而需要把参数在组件中传递两次才能最终将 A组件 中的 prop 传递给 C组件 。

> https://facebook.github.io/react/docs/context.html
> https://segmentfault.com/a/1190000005356878?utm_source=tuicool&utm_medium=referral

# `ref的值`
ref的值有两种类型，一种是字符串，一种是回调函数。
ref属性也可以是一个回调函数而不是一个名字。   这个函数将要在组件被挂载之后立即执行

# `PropTypes`
PropTypes.arrayOf()指定类型组成的数组，PropTypes.shape()指定特定属性组成的对象。

```js
TodoList.propTypes = {
  todos: PropTypes.arrayOf(PropTypes.shape({
    id: PropTypes.number.isRequired,
    completed: PropTypes.bool.isRequired,
    text: PropTypes.string.isRequired
  }).isRequired).isRequired,
  onTodoClick: PropTypes.func.isRequired
}
```
# `componentWillReceiveProps`

在组件接收到新的 props 的时候调用。在初始化渲染的时候，该方法不会调用。
用此函数可以作为 react 在 prop 传入之后， render() 渲染之前更新 state 的机会。
老的 props 可以通过 this.props 获取到。
