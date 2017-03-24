---
title: redux学习笔记
date: 2017-03-10 15:09:45
tags: [react ,redux]
categories: 技术
---

# Redux
>http://redux.js.org/?q=#
入门参考：
>http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_one_basic_usages.html

## 设计思想：    
> 1. Web 应用是一个状态机，视图和状态是一一对应的。
> 2. 所有的状态，保存在一个对象里面

## 基本概念和API   
1. Store
  Store就是保存数据的地方，可以把它看成一个容器。整个应用只能有一个Store。
  Redux提供`createStore`这个函数，用来生成Store。

  ```jsx
  import {createStore} from 'redux';
  const store=createStore(fn);
  ```
  `createStore`函数接受另一函数作为参数，返回生成的Store对象      

2. State
  `Store`对象包含所有数据。如果想要得到某个时间点的数据，就要对Store生成快照。这种时点的数据集合，就叫做State。
  当前时刻的State，可以通过store.getState()拿到。  

  ```jsx
  import {createStore} from 'redux';
  const store = createStore(fn);
  const state = store.getState();
  ```
  Redux规定，一个state对应一个view。只要state相同，view就相同。你知道state就知道view是什么样，反之亦然。

3. Action
  State的变化导致view的变化。但是用户接触不到state，只能接触到view。所以，state的变化必须是view导致的。
  action就是view发出的通知，表示state应该要发生变化了。
  action就是一个对象。其中的`type`属性是必须的，表示action的名称。其他属性可以自由设置，社区有一个[规范](https://github.com/acdlite/flux-standard-action)可以参考

  ```jsx
  const action={
    type:"ADD_TODO",
    payload:'learn Redux'
  }
  ```
  可以这样理解，Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action。它会运送数据到 Store。   

4. Action Creator
  View 要发送多少消息，就会有多少种Action。可以定义一个函数生成Action，这个函数就叫Action Creator。

  ```jsx
  const ADD_TODO = '添加 TODO';
  function addTodo(text){
    return{
      type:ADD_TODO,
      payload:text
    }
  }
  const action = addTodo('Learn Redux');
  ```
  `addTodo`函数就是一个Action Creator。  

5. store.dispatch()
  store.dispatch() 是view 发出的唯一方法。

  ```jsx
  import {createStore} from 'redux';
  const store = createStore(fn);
  store.dispatch({
    type:'ADD_TODO',
    payload:'Learn Redux'
  });
  ```
  `store.dispatch`接受一个Action对象作为参数，将它发送出去。
  结合Action Creator ，这段代码可以改写如下：
  `store.dispatch(addTodo('learn Redux'))`

6. Reducer
  store 收到Action 以后，必须给出一个新的state ，这样view才会发生变化。这种state的计算过程就叫做Reducer。
  Reducer是一个函数，它接受Action 和当前 State 作为参数，返回一个新的state。

  ```jsx
  const reducer = function (state, action) {
    //...
    return new_state;
  }
  ```

  整个应用的初始状态，可以作为State的默认值。如下例子：

  ```jsx
  const defaultState=0;
  const reducer = (state = defaultState ,action) => {
    switch (action.type){
      case 'ADD':
        return state + action.payload;
      default:
        return state;
    }
  }
  const state = reducer(1, {
    type:'ADD',
    payload:2
  });
  ```
  上面代码中，reducer函数收到名为ADD的 Action 以后，就返回一个新的 State，作为加法的计算结果。其他运算的逻辑（比如减法），也可以根据 Action 的不同来实现。
实际应用中，Reducer 函数不用像上面这样手动调用，store.dispatch方法会触发 Reducer 的自动执行。为此，Store 需要知道 Reducer 函数，做法就是在生成 Store 的时候，将 Reducer 传入createStore方法。

```jsx 
 import {createStore} from 'redux';
 const store = createStore(reducer);
```
上面代码中，createStore接受Reducer作为参数，生成一个新的Store。以后每当store.dispatch发送过来一个新的action，就会自动调用Reducer，得到新的state。
为什么这个函数叫做Reducer？因为他可以作为数组的reduce方法的参数。请看下面的例子，一系列action对象按照顺序作为一个数组。

```jsx
  const actions = [
    {type:'ADD',payload:0},
    {type:'ADD',payload:1},
    {type:'ADD',payload:2},
  ];
  const total = actions.reduce(reducer,0);//3
```
上面代码中，数组actions表示依次有三个 Action，分别是加0、加1和加2。数组的reduce方法接受 Reducer 函数作为参数，就可以直接得到最终的状态3。

7. 纯函数
  Reducer函数最重要特征是，他是一个纯函数。也就是说，只要是同样的输入，必定得到同样的输出。
  纯函数是函数编程的概念，必须遵守以下约束。

  > 不得改写参数
  > 不能调用系统 I/O 的API
  > 不能调用Date.now()或者Math.random()等不纯的方法，因为每次会得到不一样的结果

由于 Reducer 是纯函数，就可以保证同样的State，必定得到同样的 View。但也正因为这一点，Reducer 函数里面不能改变 State，必须返回一个全新的对象，请参考下面的写法。

```jsx
  //state是一个对象
  function reducer(state, action){
    return Object.assign({},state,{ thingToChange });
    //或者
    return {...state,...newState}
  }

  //state 是一个数组
  function reducer(state,action){
    return [...state,newItem];
  }
```
最好把 State 对象设成只读。你没法改变它，要得到新的 State，唯一办法就是生成一个新对象。这样的好处是，任何时候，与某个 View 对应的 State 总是一个不变的对象

8. store.subscribe()
  store 允许使用store.subscribe方法设置监听函数，一旦state发生变化，就自动执行这个函数。

  ```jsx
  import {createStore} from 'redux';
  const store = createStore(reducer);

  store.subscribe(listener);
  ```
  显然，只要把view的更新函数（对于React项目，就是组件的render方法或setState方法）放入listen，就会实现view的自动渲染。
  store.subscribe方法返回一个函数，调用这个函数就可以解除监听。

  ```jsx 
  let unsubscribe = store.subscribe(()=>{
    console.log(store.getState())
  });

  unsubscribe();
  ```
# Store的实现。
  store提供了三个方法。

  > store.getState()
  > store.dispatch()
  > store.subscribe()

  ```jsx
  import {createStore} from 'redux';
  let {subscribe,dispatch,getState} = createStore(reducer);
  ```
createStore方法还可以接受第二个参数，表示 State 的最初状态。这通常是服务器给出的。

`let store = createStore(todoApp, window.STATE_FROM_SERVER)`

上面代码中，window.STATE_FROM_SERVER就是整个应用的状态初始值。注意，如果提供了这个参数，它会覆盖 Reducer 函数的默认初始值。
下面是createStore方法的一个简单实现，可以了解一下 Store 是怎么生成的。

```jsx
  const createStore = (reducer) => {
    let state;
    let listeners=[];

    const getState=() => state;

    const dispatch = (action) => {
      state = reducer(state,action);
      listeners.forEach(listener => listener());
    };

    const subscribe = (listener) => {
      listeners.push(listener);
      return () => {
        listeners = listeners.filter(l => l !==listener);
      }
    };

    dispatch({});

    return {getState,dispatch,subscribe};

  }
```
# Reducer的拆分
  `Reducer函数负责生成state`。由于整个应用只有一个state对象，包含所有数据，对于大型应用来说，这个state必然十分庞大，导致Reducer函数也十分庞大。

  ```js
  const chatReducer = (state = defaultState, action = {}) => {
    const { type, payload } = action;
    switch(type){
      case ADD_CHAT:
        return Object.assign({},state,{
          chatLog:state.chatLog.concat(payload)
        });
      case CHANGE_STATUS:
        return Object.assign({},state,{
          statusMessage:payload
        });
      case CHANGE_USERNAME:
        return Object.assign({},state,{
          userName:payload
        });
      default:return state;
    }
  }
  ```
  上面代码中，三种 Action 分别改变 State 的三个属性。   

  > ADD_CHAT：chatLog属性
  > CHANGE_STATUS：statusMessage属性
  > CHANGE_USERNAME：userName属性
这三个属性之间没有联系，这提示我们可以把 Reducer 函数拆分。不同的函数负责处理不同属性，最终把它们合并成一个大的 Reducer 即可。

```js
  const chatReducer = (state = defaultState,action = {})=>{
    return{
      chatLog:chatLog(state.chatLog,action),
      statusMessage:statusMessage(state.statusMessage,action),
      userName:userName(state.userName,action)
    }
  };
```

上面代码中，Reducer 函数被拆成了三个小函数，每一个负责生成对应的属性。
这样一拆，Reducer 就易读易写多了。而且，这种拆分与 React 应用的结构相吻合：一个 React 根组件由很多子组件构成。这就是说，子组件与子 Reducer 完全可以对应。
Redux 提供了一个`combineReducers`方法，用于 Reducer 的拆分。你只要定义各个子 Reducer 函数，然后用这个方法，将它们合成一个大的 Reducer。

```js
  import {combineReducers} from 'redux';
  const chatReducer = combineReducers(
    chatLog,
    statusMessage,
    userName
  )
  export default todoApp;
```

上面的代码通过`combineReducers`方法将三个子 Reducer 合并成一个大的函数。
这种写法有一个前提，就是 State 的属性名必须与子 Reducer 同名。如果不同名，就要采用下面的写法。
`只要视图发来一个action，sotre会调用每一个子reducer，每一个子reducer只会更新其对应的store的属性`

```js
const reducer = combineReducers({
  a: doSomethingWithA,
  b: processB,
  c: c
})

// 等同于
function reducer(state = {}, action) {
  return {
    a: doSomethingWithA(state.a, action),
    b: processB(state.b, action),
    c: c(state.c, action)
  }
}
```
下面是`combineReducer`的简单实现。

```js
const combineReducers = reducers => {
  return (state = {},action)=>{
    return Object.keys(reducers).reduce(
      (nextState,key) => {
        nextState[key] = reducers[key]( state[key] , action);
        return newxtState;
      },
      {}
    );
  };
};
```

可以吧所有的子Reducer放在一个文件里，然后统一引入。

```js
  import {combineReducers} from 'redux'
  import * as reducers from './reducers'

  const reducer = combineReducers(reducers)
```

## 工作流 
  Redux的工作流程梳理：
  [Redux Flow](./Redux-flow.png)

首先，用户发出 Action。

`store.dispatch(action);`

然后，Store 自动调用 Reducer，并且传入两个参数：当前 State 和收到的 Action。 Reducer 会返回新的 State 。

`let nextState = todoApp(previousState, action);`

State 一旦有变化，Store 就会调用监听函数。

// 设置监听函数
`store.subscribe(listener);`

listener可以通过store.getState()得到当前状态。如果使用的是 React，这时可以触发重新渲染 View。

```js
function listerner() {
  let newState = store.getState();
  component.setState(newState);   
}
```
# Redux 入门教程（二）：中间件与异步操作
> http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html

    一个关键问题没有解决：异步操作怎么办？Action 发出以后，Reducer 立即算出 State，这叫做同步；Action 发出以后，过一段时间再执行 Reducer，这就是异步。
怎么才能 Reducer 在异步操作结束后自动执行呢？这就要用到新的工具：中间件（middleware）。

## 中间件的概念   

  为了理解中间件，让我们站在框架作者的角度思考问题：如果要添加功能，你会在哪个环节添加？

```js
 1. Reducer:纯函数，只承担计算State的功能，不适合承担其他功能，也承担不了，因为理论上，纯函数不能进行读写操作。
 2. View：与State 一一对应，可以看做State的视觉层，也不适合承担其他功能。
 3. Action：存放数据的对象，即消息的载体，只能被别人操作，自己不能进行任何操作。
```
想来想去，只有发送Action的这个步骤，即`store.dispatch()`方法，可以添加功能。举例来说，要添加日志功能，把Action和State打印出来，
可以对`store.dispatch`进行如下改造。

```js
 let next = store.dispatch;
 store.dispatch = function dispatchAndLog(action){
   console.log('dispatching',action);
   next(action);
   console.log('next state',store.getState());
 }
```
## 中间件的用法
本教程不涉及如何编写中间件，因为常用的中间件都有现成的，只要引用别人写好的模块即可。
比如，上一节的日志中间件，就有现成的[redux-logger](https://github.com/evgenyrodionov/redux-logger)模块。这里只介绍怎么使用中间件。

```jsx
  import {applyMiddleware,createStore} from 'redux';
  import createLogger from 'redux-logger';
  const logger = createLogger();

  const store = createStore(
    reducer,
    applyMiddleware(logger)
  );
```
上面代码中，`redux-logger`提供一个生成器`createLogger`,可以生成日志中间件`logger`。然后，将它放在`applyMiddleware`方法中，传入`createStore`方法，
就完成了`store.dispatch()`的功能增强。
这里有两点需要注意：

1. `createStore` 方法可以结接收整个应用的初始状态作为参数，那样的话，`applyMiddleware`就是第三个参数了。

```jsx
 const store = createStore(
   reducer,
   initial_state,
   applyMiddleware(logger)
 )
```

2. 中间件的次序有讲究

```js
 const store = createStore(
   reducer,
   applyMiddleware(thunk,promise,logger)
 )
``` 

上面代码中，`applyMiddleware`方法的三个参数，就是三个中间件。有的中间件有次序要求，使用前要查一下文档。比如，`logger`就一定要放在最后，否则输出结果会不正确。

## applyMiddlewares()
看到这里，你可能会问，`applyMiddlewares`这个方法到底是干什么的？
它是 Redux 的原生方法，作用是将所有中间件组成一个数组，依次执行。下面是它的源码。

```js
export default function applayMiddleware(...middlewares){
  return (createStore) => (reducer,preloadedState,enhancer)=>{
    var store = createStore(reducer,preloadedState,enhancer);
    var dispatch = store.dispatch;
    var chain = [];
    var middlewareAPI = {
      getState:store.getState,
      dispatch:(action)=>dispatch(action)
    };
    chain = middlewares.map(middleware => middleware(middlewareAPI));
    dispatch = compose(...chain)(store.dispatch);

    return {...store,dispatch}
  }
}
```
上面代码中，所有中间件被放进了一个数组chain，然后嵌套执行，最后执行store.dispatch。可以看到，中间件内部（middlewareAPI）可以拿到getState和dispatch这两个方法。

## 异步操作的基本思路

理解了中间件以后，就可以处理异步操作了。
同步操作只要发出一种Action即可，异步操作的差别就是他要出三种Action。
 
 > - 操作发起时的 Action
 > - 操作成功时的 Action
 > - 操作失败时的 Action
 以向服务器获取数据为例，三种Action可以两种不同的写法。

 ```js
 //写法一： 名相同，参数不同
 { type: 'FETCH_POSTS' }
 { type: 'FETCH_POSTS', status: 'error', error: 'Oops' }
 { type: 'FETCH_POSTS', status: 'success', response: { ... } }

 //写法二： 名称不同
 { type: 'FETCH_POSTS_REQUEST' }
 { type: 'FETCH_POSTS_FAILURE', error:"Oops"}
 { type: 'FETCH_POSTS_SUCCESS', response: {...}}
 ```
除了Action种类不同，异步操作的State也要进行改造，反应不同的操作状态。下面是State的例子。

```js
  let state = {
    //....
    isFetching:true,
    didInvalidate:true,
    lastUpdated:'xxxxxx'
  };
```
上面代码中，State属性isFetching表示是否在抓取数据。didInvalidate表示数据是否过时，lastUpdate表示上一次更新时间。
现在，整个异步操作的思路就很清楚了。

 - 操作开始时，发出一个Action，触发State更新为“正在操作”状态，view重新渲染。
 - 操作结束后，再送出一个Action，触发State更新为‘操作结束’状态，view再一次重新渲染。

## redux-thunk 中间件
异步操作至少要送出两个Action：用户触发第一个Action，这跟同步操作一样，没有问题。如何才能在操作结束时，系统自动送出第二个Action呢？
奥妙就在Action creator中。

```js
 class AsyncApp extends Component{
   componentDidMount() {
     const { dispatch, selectedPost } = this.props;
     dispatch(fetchPosts(selectedPost));
   }
 }
```
上面代码是一个异步组件的例子。加载成功后（componentDidMount方法），它送出了（dispatch方法）一个Action，向服务器要求数据
`fetchPosts(selectedSubreddit)`。这里fetchPosts就是Action Creator。
下面就是fetchPosts的代码，关键之处就在里面。

```js
//Async Action  Example

import fetch from 'isomorphic-fetch';
export function fetchFriends(){
  return dispatch => {
    dispatch({type: "FETCH_FRIENDS" });
    return fetch('http://localhost/api/friends')
      .then(response => response.json())
      .then(json =>{
        dispatch({type: 'RECEIVE_FRIENDS',payload:json});
      })
  }
}
```

```js
  const fetchPosts = postTitle => (dispatch,getState)=>{
    dispatch(requestPosts(postTitle));
    return fetch(`/some/API/${postTitle}.json`)
      .then(response => response.json())
      .then(json => dispatch(recervePosts(postTitle,json));
  }
  //使用方法一
  store.dispatch(fetchPosts('reactjs'))
  //使用方法二
  store.dispatch(fetchPosts('reactjs')).then(() => {
    console.log(store.getState)
  })
```
上面代码中，`fetchPosts`是一个Action Creator（动作生成器），返回一个函数。这个函数执行后，先发出一个Action（`requestPosts(postTitle)`）,
然后进行异步操作，拿到结果后，先将结果转化为JSON格式，然后再发出一个Action（`receivePosts(postTitle,json)`）。

 1. fetchPosts返回了一个函数，而普通Action Creator默认返回一个对象。
 2. 返回的函数的参数是dispatch和getState这两个Redux方法，普通的Action Creator的参数是Action内容
 3. 返回的函数中，先发出一个Action（requestPosts(postTitle)），表示操作开始。
 4. 异步操作结束后，再发出一个Action（receivePost(postTitle,json)）,表示操作结束。
 这样的处理，就解决了自动发送第二个Action 的问题。但是又带来了一个新的问题，Action是由store.dispatch发送的。而store.dispatch方法正常情况下，
 参数只能是对象，而不能是函数。这时，就要使用中间件[redux-thunk](https://github.com/gaearon/redux-thunk)。

 ```js
  import {createStore, applyMiddleware} from 'redux'
  import thunk from 'redux-thunk'
  import reducer from './reducers'

 //Note: this  api requires redux@>=3.1.0
  const store= createStore(
    reducer,
    applyMiddleware(thunk)
  )
 ```
上面代码使用redux-thunk中间件，改造store.dispatch，使得后者可以接受函数作为参数。
因此，异步操作的第一种解决方案就是，写出一个返回函数的 Action Creator，然后使用redux-thunk中间件改造store.dispatch。

## redux-promise中间件
 既然Action Creator可以返回函数，当然也可以返回其它值。另一种异步操作的解决方案，就是让Action Creator 返回一个Promise对象。
 这就需要使用redux-promise中间件。

 ```js
 import { createStore, applyMiddleware } from 'redux';
 import promiseMiddleware from 'redux-promise';
 import reducer from './reducers';

 const store = createStore(
   reducer,
   applyMiddleware(promiseMiddleware)
 ) ;
 ```

 这个中间件是的`store.dispatch`方法可以接受Promise对象作为参数。这时，Action Creator有两种写法。
 写法一，返回值是一个Promise对象。

 ```js
 const fetchPosts = (dispatch, postTitle) => new Promise(function (resolve,reject){
   dispatch(requestPosts(postTitle));
   return fetch(`/some/API/${postTitle}.json`)
      .then(response => {
        type: 'FETCH_POSTS',
        payload: response.json()
      });
 });

 ```

 写法二，Action对象的payload属性是一个Promise对象。这需要从`redux-actions`模块引入createAction方法，并且写法也要变成下面这样。

 ```js
 import { createAction } from 'redux-actions';
 class AsyncApp extends Component {
   componentDidMount(){
     const { dispatch , selectedPost } = this.props;
     //发出同步Action
     dispatch(requestPosts(selectedPost));
     //发出异步Action
     dispatch(createAction(
       'FETCH_PSOTS',
       fetch(`/some/API/${postTitle}.json`)
          .then(response=>response.json())
     ));
   }
 }
 ```
 上面代码中，第二个`dispatch`方法发出的是异步Action，只有等到操作结束，这个Action才会实际发出。
 注意，createAction的 第二个参数必须是一个Promise对象。
 看一下`redux-promise`的[源码](https://github.com/acdlite/redux-promise/blob/master/src/index.js)，就会明白它内部是如何操作的。

```js
  export default function promiseMiddleware({ dispatch }){
    return next => action => {
      if(!isFSA(action)){
        return isPromise(action)
          ? action.then(dispatch)
          : next(action);
      }
      return isPromise(action.payload)
        ? action.payload.then(
          result => dispatch({...action,payload: result }),
          error =>{
            dispatch({...action,payoad:error,error:true});
            return Promise.reject(error);
          }
        )
        : next(action);
    };
  };
```
从上面代码可以看出，如果Action本身是一个Promise，它resolve以后的值应该是一个Action，会被
`dispatch`方法送出（`action.then(dispatch)`）,但reject以后不会有任何动作；如果Action对象
的`payload`属性是一个Promise对象，那么无论resolve和reject，dispatch方法都会发出Action。
中间件和异步操作，就介绍到这里。

# React-Redux 的用法
>http://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_three_react-redux.html
## UI组件
React-Redux将所有组件分成两大类：UI组件（presentational component）和容器组件（container component）

UI组件有以下几个特征。
 - 只负责UI的呈现，不带有任何业务组件
 - 没有状态（即不使用this.state这个变量）
 - 所有数据都由参数（this.props）提供
 - 不使用任何Redux的API

 下面就是一个UI组件的例子。

 ```js
const Title=value => <h1>{value}</h1>;
 ```
 因为不含有状态，UI组件又称为“纯组件”，即纯函数一样，纯粹由参数决定它的值。

##容器组件
容器组件特征恰恰相反
-  负责管理数据和业务逻辑，不负责 UI 的呈现。
- 带有内部状态state
- 使用Redux的API

总之，UI组件负责UI的呈现，容器组件负责管理数据和逻辑。
如果一个组件既有UI又有业务逻辑，则将它拆分成下面的结构：外面是一个容器组件，里面包含了一个UI组件。前者负责与外部的通讯，将数据传给后者，由后者渲染出视图。
React-Redux规定，所有的UI组件都由用户提供，容器组件则是由React-Redux自动生成。也就是说，用户负责视觉层，状态管理则是全部交给它。

## connect()
React-Redux提供connect方法，用于从UI组件生成容器组件。connect的意思，就是将这两种组件连起来。

```js
 import {connect} from 'react-redux'
 const VisibleTodoList = connect()(TodoList);
```
上面代码中，TodoList是UI组件，VisibleTodoList就是由React-Redux通过connect方法自动生成的容器组件。
但是，因为没有定义业务逻辑，上面这个容器组件毫无意义，只是UI组件的一个单纯的包装层。为了定义业务逻辑，需要给出下面两方面的信息。

1. 输入逻辑：外部的数据（即state对象）如何转换为UI组件的参数
2. 输出逻辑：用户发出的动作如何变为 Action 对象，从UI组件传出去

因此，connect方法的完整API如下。

```js
 import { connect } from 'react-redux'
 const VisibleTodoList = connect(
   mapStateToProps,
   mapDispatchToProps,
 )(TodoList)
```
上面代码中，connect 方法接受两个参数：mapStateToProps和mapDispatchToProps。它们定义了UI组件的业务逻辑。
前者负责输入逻辑，即将state映射到UI组件的参数（props），后者负责输出逻辑，即将用户对UI组件的操作映射成Action。

## mapStateToProps()
`mapStateToProps`是一个函数。它的作用就是像它的名字那样，建立一个从（外部的）state对象到（UI组件的）props对象的映射关系。
作为函数，mapStateToProps执行后应该返回一个对象，里面的每一个键值对就是一个映射。请看下面的例子

```js
 const mapStateToProps = (state) =>{
   return {
     todos:getVisibleTodos(state.todos,state.visibilityFilter)
   }
 }
```
上面代码中，`mapStateToProps`是一个函数，它接受`state`作为参数，返回一个对象。这个对象有一个todos属性，代表UI组件的同名参数，
后面的`getVisibleTodos`也是一个函数，可以从`state`算出`todos`。
下面就是`getVisibleTodos`的一个例子，用来计算出`todos`。

```js
 const getVisibleTodos = (todos,filter) => {
   switch(filter){
     case 'SHOW_ALL':
      return todos;
     case 'SHOW_COMPLETED':
      return todos.filter(t=>t.completed)
     case 'SHOW_ACTIVE':
      return todos.filter(t => !t.completed)
     default:
     throw new Error('unknow filter:'+ filter)
   }
 }
```
`mapStateToProps`会订阅Store，每当`state`更新的时候就会自动执行，重新计算UI组件的参数，从而触发UI组件的重新渲染。
`mapStateToProps`的第一个参数总是`state`对象，还可以使用第二个参数，代表容器组件的props对象。

```js
  //容器组件的代码
  //<FilterLink filter="SHOW_ALL">
  // ALL
  //</FilterLink>
  const mapStateToProps = (state, ownProps) => {
    return {
      active: ownProps.filter === state.visibilityFilter
    }
  }
```
使用`ownProps`作为参数后，如果容器组件的参数发生变化，也会引发UI组件重新渲染。
`connect`方法可以省略`mapStateToProps`参数，那样的话，UI组件就不会订阅Store，就是说Store的更新不会引起UI组件的更新。
##  mapDispatchToProps()
`mapDispatchToProps`是`connect`函数的第二个参数，用来建立UI组件的参数到`store.dispatch`方法的映射。也就是说，它定义了哪些
用户的操作应当作Action，传给Store。它可以是一个函数，也可以是一个对象。
如果`mapDispatchToProps`是一个函数，会得到`dispatch`和`ownProps`(容器组件的`props`对象)两个参数。

```js
const mapDispatchToProps = (
  dispatch,
  ownProps
) => {
  return {
    onClick:() => {
      dispatch({
        type:'SET_VISIBILITY_FILTER',
        filter:ownProps.filter
      });
    }
  };
}
```
上面代码可以看到，`mapDispatchToProps`作为函数，应该返回一个对象，该对象的每一个键值对都是一个映射，定义了UI组件的参数怎样发出Action。
如果`mapDispatchToProps`是一个对象，它的每一个建名也是对应UI组件的同名参数，键值应该是一个函数，会被当做Action Creator，返回的Action
会由Redux自动触发，举例来说，上面的`mapDispatchToProps`写成对象就是下面这样。

```js
  const mapDispatchToProps = {
    onClick:(filter) =>{
      type:'SET_VISIBILITY_FILTER',
      filter:filter
    }
  }
```
## <Provider>组件

`connect`方法生成容器组件以后，需要让容器组件拿到`state`对象，才能生成UI组件的参数。
一种解决方法是将`state`对象作为参数，传入容器组件。但是，这样做比较麻烦，尤其是容器组件可能在很深的层级，一级级将`state`传下去就很麻烦。
React-redux提供`Provider`组件，可以让容器组件拿到`state`

```js
  import { Prpvider } from 'react-redux'
  import { createStore } from 'redux'
  import todoApp  from './reducers'
  import App from './components/App'

  let store = createStore(todoApp);

  render(
    <Provider store={store}>
      <App/>
    </Provider>,
    document.getElementById('root')
  )
```
上面代码中，`Provider`在根组件外面包了一层，这样一来，`App`的所有子组件就默认都可以拿到`state`了。
它的原理是`React`组件的`context`属性，请看源码。

```js
  class Provider extends Component{
    getChildContext(){
      return{
        store:this.props.store
      };
    }
    render(){
      return this.props.children;
    }
  }
  Provider.childContextTypes = {
    store:React.PropTypes.object
  }
```
上面代码中，`store`放在了上下文对象`context`。然后，子组件就可以从`context`拿到`store`，代码大致如下。

```js
class VisibleTodoList extends Component {
  componentDidMount() {
    const { store } = this.context;
    this.unsubscribe= store.subscribe(() =>
      this.forceUpdate()
    );
  }
  render(){
    const props = this.props;
    const { store } = this.context;
    const state = store.getState;
    //....
  }
}
VisibleTodoList.contextTypes = {
  store: React.PropsTypes.object
}
```
`React-Redux`自动生成的容器组件代码，就类似上面这样，从而拿到`store`.
## 实例：计数器
下面是一个计数器组件，它是一个纯的UI组件。

```js
class Counter extends Component {
  render(){
    const { value, onIncreaseClick } = this.props
    return (
      <div>
        <span>{vlaue}</span>
        <button onClick = {onIncreaseClick}>Increase</button>
      </div>
    )
  }
}
```
上面代码中，这个 UI 组件有两个参数：value和onIncreaseClick。前者需要从state计算得到，后者需要向外发出Action。
接着，定义`value`到`state`的映射，以及`onIncreaseClick`到`dispatch`的映射。

```js
function mapStateToProps(state) {
  return {
    value:state.count
  }
}
function mapDispatchToProps(dispatch) {
  return {
    onIncreaseClick:() => dispatch(increaseAction)
  }
}
//Action Creator
const increaseAction = { type: 'increase' };
```
然后，使用`connect`方法生成容器组件。

```js
 const App = connect(
   mapStateToProps,
   mapDispatchToProps
 )(Counter)
```

然后定义这个组件的Reducer。

```js
//reducer
function counter(state = {count:0 }, action){
  const count = state.count
  switch(action.type){
    case 'increase':
      return { count:count +1 }
    default:
      return state
  }
}
```
最后，生成`store`对象，并使用`Provider`在根组件外面包一层。

```js
import { loadState, saveState } from './localStorage';

const persistedState = loadState();
const store = createStore(
  todoApp,
  persistedState
);

store.subscribe(throttle(()=>{
  saveState({
    todos:store.getState().todos,
  })
},1000))

ReactDOM.render(
  <Provider store = {store}>
    <App/>
  </Provider>,
  document.getElementById('root')  
);
```
完整代码看[这里](https://github.com/jackielii/simplest-redux-example/blob/master/index.js).

## React-Router 路由库
使用`React-Router`的项目，与其他项目没有不同之处，也是使用`Provider`在`Router`外面包一层，毕竟`Provider`的唯一功能就是传入`store`对象。

```js
  const Root = ({ store }) =>(
    <Provider store={ store }>
      <Router>
        <Route path='/' component={App}/>
      </Router>
    </Provider>
  );
```
