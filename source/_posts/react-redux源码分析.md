---
title: react-redux源码分析
date: 2017-05-01 19:19:32
tags: [react-redux]
categories: 源码分析
---
阅读感想：
1. 阅读源码前，必须对项目对外暴露的API，有大致的了解。
2. 有了上面的基础，开始从入口，进行阅读。
3. 阅读常用的API实现。
4. 阅读的时候，要猜。由功能猜实现，对源码进行理解。

> 本文是对react-redux@5.0.4的源码分析,阅读中添加了注释的代码[react-redux-analysis](https://github.com/sivkun/react-redux-analysis)

react-redux的API参考 [链接](https://github.com/reactjs/react-redux/blob/master/docs/api.md#connectmapstatetoprops-mapdispatchtoprops-mergeprops-options)

react-redux在5.0中做出了很大改动。 [请参考](https://github.com/reactjs/react-redux/pull/416)

作者为了react-redux在实际项目中可定制化，重写了`connect`,分成了很多模块组件。
让`connect`作为了`connectAdvanced`的门面（外观模式）。

更新后的文件结构如下：
![react-redux文件结构](react-redux源码分析\react-redux结构.png)

<!-- more -->

# Provider
provider组件包裹整个APP，通过Props将store传入。使的通过connect()(component)生成的容器组件，可以获取到store。
Provider使用方法
```js
ReactDOM.render(
  <Provider store={store}>
    <MyRootComponent />
  </Provider>,
  rootEl
)
```
如何实现？

Provider组件通过使用react提供的顶层API `context`特性实现。
`getChildContext`
```js
getChildContext() {
    return { store: this.store, storeSubscription: null }
  }
```
`childContextTypes`
```js
Provider.childContextTypes = {
  store: storeShape.isRequired,
  storeSubscription: subscriptionShape
}
```
这样，通过connect()(wrappedcomponent)生成的容器组件中，可以通过设置`contextTypes`,
在组件内部可以通过`this.context.store`获取到store,对状态进行操作。
```js
Component.contextTypes = {
  store: storeShape.isRequired,
};
```
# connect
首先通过react-redux的链接充分理解connect的使用。
`connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])`
作用是，在Redux store和React Component之间建立起联系。
`connect`方法是`connectAdvanced`的门面（外观模式）,为大多数情况下的使用，提供便利的API，
也就是说`connect`方法简化了`connectAdvanced`的API。
`connect`的具体接口如下：
```js
 connect(
    mapStateToProps(state,ownProps)=>stateProps:Object, 
    mapDispatchToProps(dispatch, ownProps)=>dispatchProps:Object, 
    mergeProps(stateProps, dispatchProps, ownProps)=>props:Object,
    options:Object
   )=>(
     component
   )=>component
```
## 可以做出如下猜想判断：
1. 使用该方法后，返回一个包裹原先定义的xxxComponent的新的newReactComponent
2. connect方法执行后返回`wrapWithConnect`函数，在其内部形成一个闭包，保存了传入的mapToProps(选择器)等信息。
  并且执行该函数后，返回包裹后的newReactComponent，而该组件通过render原组件，形成对原组件的封装。
3. 渲染页面需要store tree（通过`context`获取`provider`中的`store`）中的`state`片段，变更`state`需要`dispatch action`，这两处信息，就是在调用connect时，作为参数传入的`mapStateToProps`函数和`mapDispatchToProps`函数，这两个函数过滤`state`和`ownprops`后生成的`props`，connect包裹后形成的newReactComponent组件通过设置原组件的props属性传入信息到原组件。

**选择器**
更多[参考reselect](https://github.com/reactjs/reselect)
`mapStateToProps`
顾名思义该函作用是 state映射生成Props，可以看做是一个selector
react-redux内部调用该函数，并把state作为参数传入
`mapDispatchToProps`
顾名思义该函作用是 dispatch映射生成Props，可以看做是一个selector
在函数定义时，引用外部自定义action，在react-redux内部调用该函数，并把store.dispatch作为参数传入

以上都是通过react-rdux最常用的方式以及API可以分析出的结果。

## 具体实现 connect.js

```js
export function createConnect({
  connectHOC = connectAdvanced,
  mapStateToPropsFactories = defaultMapStateToPropsFactories,
  mapDispatchToPropsFactories = defaultMapDispatchToPropsFactories,
  mergePropsFactories = defaultMergePropsFactories,
  selectorFactory = defaultSelectorFactory
} = {}) {
  return function connect(
    mapStateToProps,
    mapDispatchToProps,
    mergeProps,
    {
      pure = true,
      areStatesEqual = strictEqual,
      areOwnPropsEqual = shallowEqual,
      areStatePropsEqual = shallowEqual,
      areMergedPropsEqual = shallowEqual,
      ...extraOptions
    } = {}
  ) {
    //结合selectorFactory.js中函数finalPropsSelectorFactory，
    //initXXX的signature是(dispatch,option)=>mapXXXToProps(XXX,ownProps)=>XXXProps，
    //initXXX执行后，就是初始化了参数的XXX函数。
    //可以给mapXXXToProps函数设置dependsOnOwnProps属性，表示是否使用ownProps属性
    const initMapStateToProps = match(mapStateToProps, mapStateToPropsFactories, 'mapStateToProps')
    const initMapDispatchToProps = match(mapDispatchToProps, mapDispatchToPropsFactories, 'mapDispatchToProps')
    const initMergeProps = match(mergeProps, mergePropsFactories, 'mergeProps')
    return connectHOC(selectorFactory, {
      // used in error messages
      methodName: 'connect',
       // used to compute Connect's displayName from the wrapped component's displayName.
      getDisplayName: name => `Connect(${name})`,

      // if mapStateToProps is falsy, the Connect component doesn't subscribe to store state changes
      shouldHandleStateChanges: Boolean(mapStateToProps),

      // passed through to selectorFactory
      initMapStateToProps,
      initMapDispatchToProps,
      initMergeProps,
      pure,
      areStatesEqual,
      areOwnPropsEqual,
      areStatePropsEqual,
      areMergedPropsEqual,

      // any extra options args can override defaults of connect or connectAdvanced
      ...extraOptions
    }) //结果返回wrapWithConnect(WrappedComponent)=>newWrapedComponent
  }
}

export default createConnect()
```
可以看出`connect`函数是通过`createConnect`函数创建。
`createConnect`在声明的时候就提供了很多默认参数，内部的`connect`可以使用这些默认参数。
`connect`在声明的时候也提供了很多默认参数，`mapStateToProps`,`mapDispatchToProps`,`mergeProps`,`extraOptions`
这些参数是要在`connect`被调用的时候，由使用者提供的。当然也都可以省略。
执行`connect(...args)`最后返回的是`connectHOC(...args)`。 而`connectHOC = connectAdvanced`，
可知执行`connectAdvanced(...args)`返回函数`(wrappedcomponent)=>newComponent`;
其实这个新组件就是connectAdvanced.js中定义的`Connect`组件（整合了wrappedComponent的props）

## 具体实现connectAdvanced.js

首先看`connectAdvanced`的signature`(selectorFactory,{...options})=>(YourComponent)=>ConnectComponent`,
使用方法如下：

```js
connectAdvanced((dispatch, options) => (state, props) => ({
        thing: state.things[props.thingId],
        saveThing: fields => dispatch(actionCreators.saveThing(props.thingId, fields)),
      }))(YourComponent)
```

在connectAdvanced方法内部定义了要返回的ConnectComponent，构造函数部分代码如下：
```js
 // ...
//  ...
class Connect extends Component {
      constructor(props, context) {
        super(props, context)

        this.version = version
        this.state = {}
        this.renderCount = 0
        //获取store的方式：
        //1.通过props传递store，<connectedComponent store={innerStore}>
        //2.通过context获取store，<Provider store={outStore}>
        //通过props获取store的方式优先级更高
        this.store = props[storeKey] || context[storeKey]
        this.propsMode = Boolean(props[storeKey])
        this.setWrappedInstance = this.setWrappedInstance.bind(this)

        invariant(this.store,
          `Could not find "${storeKey}" in either the context or props of ` +
          `"${displayName}". Either wrap the root component in a <Provider>, ` +
          `or explicitly pass "${storeKey}" as a prop to "${displayName}".`
        )
        //初始化selector，selector的作用是：通过过滤state和ownProps生成，生成wrappedComponent的props。
        this.initSelector()
        //初始化subscription，subscription作用：监听state和ownProps的变化，触发wrappedComponent更新。
        this.initSubscription()
      }
  //    ...
  //    ...
}
```
### 如何生成wrappedComponent组件的props的？

生成props主要是通过selector函数，在构造函数中通过`this.initSelector()`初始化了`this.selector`对象。
代码分析：
```js
function makeSelectorStateful(sourceSelector, store) {
  // wrap the selector in an object that tracks its results between runs.
  const selector = {
    run: function runComponentSelector(props) {
      try {
        //此处看出nextProps由sourceSelector函数传入state和props计算得来。
        const nextProps = sourceSelector(store.getState(), props)
        if (nextProps !== selector.props || selector.error) {
          selector.shouldComponentUpdate = true
          selector.props = nextProps
          selector.error = null
        }
      } catch (error) {
        selector.shouldComponentUpdate = true
        selector.error = error
      }
    }
  }
  return selector
}
class Connect extends Component {
  initSelector() {
    //selectorFactory生成sourceSelector。
    const sourceSelector = selectorFactory(this.store.dispatch, selectorFactoryOptions)
    //this.selector具有记忆功能，如果通过sourceSelector计算后的nextProps和现在的this.props不同，则说明组件需要更新。
    this.selector = makeSelectorStateful(sourceSelector, this.store)
    this.selector.run(this.props)
  }
  //收到新的props时调用
  componentWillReceiveProps(nextProps) {
    this.selector.run(nextProps)
  }
  //通过this.selector.shouldComponentUpdate判断组件是否需要更新，优化性能。
  shouldComponentUpdate() {
    return this.selector.shouldComponentUpdate
  }
}
```
react生命周期[参考](https://facebook.github.io/react/docs/react-component.html)
初始化挂载阶段
* constructor()
* componentWillMount()
* render()
* componentDidMount()

更新阶段
* componentWillReceiveProps()
* shouldComponentUpdata()
* componentWillUpdate()
* render()
* componentDidUpdate()

卸载阶段
* componentWillUnmount()

### 返回的`ConnectComponent`是如何渲染`wrappedComponent`的？

上源码
```js
render() {
  const selector = this.selector
  selector.shouldComponentUpdate = false

  if (selector.error) {
    throw selector.error
  } else {
    return createElement(WrappedComponent, this.addExtraProps(selector.props))
  }
}
```
从代码可以看出，在`ConnectComponent`的`render`函数中，使用`createElement`创建了`WrappedComponent`也就是`YourComponent`，
并且添加了额外的属性`this.addExtraProps(selector.props)`。

### `ConnectComponent`是如何监听store变化，然后决定重新渲染的？

我们知道当react组件的state或者props改变，组件就会重新渲染，
react提供的方法有`setState`和`forceUpdate`这两个方法。
可以按照这两个线索寻找。
主要代码如下：
```js
class Connect extends Component {
  constructor(){
    ...
    this.initSubscription()
  }
  initSubscription() {
    if (!shouldHandleStateChanges) return

    // parentSub's source should match where store came from: props vs. context. A component
    // connected to the store via props shouldn't use subscription from context, or vice versa(反之亦然).
     //如果组件通过props连接store，则不应该从context获取subscription。
    const parentSub = (this.propsMode ? this.props : this.context)[subscriptionKey]
    this.subscription = new Subscription(this.store, parentSub, this.onStateChange.bind(this))

    // `notifyNestedSubs` is duplicated to handle the case where the component is  unmounted in
    // the middle of the notification loop, where `this.subscription` will then be null. An
    // extra null check every change can be avoided by copying the method onto `this` and then
    // replacing it with a no-op on unmount. This can probably be avoided if Subscription's
    // listeners logic is changed to not call listeners that have been unsubscribed in the
    // middle of the notification loop.
    this.notifyNestedSubs = this.subscription.notifyNestedSubs.bind(this.subscription)
  }
  onStateChange() {
    this.selector.run(this.props)

    if (!this.selector.shouldComponentUpdate) {
      this.notifyNestedSubs()
    } else {
      this.componentDidUpdate = this.notifyNestedSubsOnComponentDidUpdate
      this.setState(dummyState)
    }
  }
}
```
其中这段代码`this.subscription = new Subscription(this.store, parentSub, this.onStateChange.bind(this))`
作用是`store`的变化，就会触发`onStateChange`，这个函数调用会触发`this.setState`。这样`ConnectComponent`组件更新，
触发该组件render，同时子组件`YourComponent`即`wrappedComponent`的更新也会被触发，
为了提高渲染效率， `ConnectComponent`使用了`shouldComponentUpdate`判断，是否执行render，
如果返回true，不用触发render，则子组件也不必更新。

### Component更新流程

结合utils/Subscription.js源码，可以理解整个系统中组件的更新流程。
如图：
![react-redux-subscription](react-redux源码分析\react-redux-subscription.png)










