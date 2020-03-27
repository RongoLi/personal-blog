---
title: Redux之redux-thunk异步机制分析
author: Rongo Li
categories: 前端
tags:
  - react
  - redux
  - redux-thunk
date: 2019-06-9 10:12:00
---

### 前言
写该文档的目的是为了帮助刚接触react的developer快速了解react中的异步机制，通过对redux-thunk的源码分析能够加深你对action => `others` => reducer => state的理解。阅读此文档前，你需要对redux有一定的了解，可参考：[Redux中文文档](http://cn.redux.js.org/)or[Redux官方文档](https://redux.js.org/introduction/getting-started)

### redux-thunk 是什么？
redux-thunk是一个提供异步编程能力的中间件，通常改变state的方式：
```js
action => reducer => state
```
这一系列的改变过程都是同步进行的，而这远远满足不了我们的实际开发需求，例如我们需要做如下的改变：
```diff
- action => reducer => state
+ action => fetch => reducer => state
```
在`action`到达`reducer`这个过程中完成网络请求，然后将请求的数据通过`reducer`更新`state`的数据。而`redux-thunk`就是提供这种异步的能力。

### redux-thunk 怎么做到呢？
要想弄清楚它的来龙去脉，首先要看一下它的源码
```js
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```
源码看上去非常精简,首先`createThunkMiddleware(extraArgument)`接收一个`extraArgument`参数，提供给developer自己管理额外的数据。该函数返回一个接收`{dispatch,getState}`对象的匿名函数，这里我将会在应用该中间件的地方详细解释这些参数是什么和怎么传递的。再看下一步，该匿名函数又返回一个接收参数为`next`的函数。继续下一步该函数又返回一个接收参数为`action`的函数，该函数内部判断`action`的类型为函数的时候，执行该`action`函数，否则该`action`作为`next`的参数执行`next`函数。

如上就是核心部分的定义过程，该模块导出的thunk是执行`createThunkMiddleware()`后的函数，即返回接收`{ dispatch, getState }`对象的匿名函数，该返回函数对象扩充了一个`withExtraArgument`属性，并将该函数原型赋值给该属性，developer如果想做额外的数据管理可以通过`thunk.withExtraArgument('你的额外数据')`。以上就是redux-thunk的实现过程。

### redux-thunk 如何工作的？
要弄清楚redux-thunk的工作过程，我们要结合一下它的使用场景分析，如下
```js
import thunkMiddleware from 'redux-thunk';
import {createStore,applyMiddleware,compose} from 'redux';
import Immutable from 'immutable';
import rootReducer from './../Reducer/rootReducer';

const createStoreWithMiddleware = applyMiddleware(thunkMiddleware)(createStore);
export default function store() {
  const initialState = Immutable.Map();
  const store = createStoreWithMiddleware(rootReducer, initialState);
  return store;
}
```
我们看到`applyMiddleware`处使用了`redux-thunk`，那么它内部是如何工作的呢？接下来我们需要看一下redux内部的`applyMiddleware`函数的源码实现：
```js
export default function applyMiddleware(...middlewares) {
  return createStore => (...args) => {
    const store = createStore(...args)
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}
```
接下来我们分析一下它是如何使用redux-thunk中间件的，首先`applyMiddleware(...middlewares)`接收中间件并返回一个接收`createStore`的函数，这一步在上面使用场景中有调用`applyMiddleware(thunkMiddleware)(createStore)`传进来的`middlewares`就是来自`redux-thunk`的`thunkMiddleware`，而`createStore`即来自`redux`。继续下一步，该函数又返回接收`...args`的函数，该部分的调用是在`createStoreWithMiddleware(rootReducer, initialState);`,该函数内部创建了一个`store`实例，并将`store.getState`和`dispatch`函数保存在`middlewareAPI`对象中，在`middlewares.map(...)`这一步，表示遍历所有的中间件并执行该中间件函数，所以我们的redux-thunk就是其中之一，执行部分如下：
```js
return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
```
传入的参数`{ dispatch, getState }`就是middlewareAPI对象。接下来你肯定好奇next是什么了，再看上面的`dispatch = compose(...chain)(store.dispatch)`，`compose`函数将所有的中间件聚合后传入`store.dispatch`参数，所以`next`就是`store.dispatch`。最终增强版的`dispatch`即为如下函数：
```js
dispatch = (action) => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    return next(action);
  };
```
当action为普通的对象类型时：执行`next(action)`，即`store.dispatch`。当为函数类型时：将`dispatch,getState,extraArgument`作为参数传入并执行该函数，回顾一下这些参数的来源。
```js
const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }

function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }

    return next(action);
  };
}
```
再看一下函数类型的的`action`使用场景
```js
export function fetchData (params) {
  return dispatch => {
    services.fetchComments(params)
      .then(res => {
        dispatch({type:'actionType',data:res});
      });
  };
}
```

### 注意
需要注意的是`middlewareAPI`中`dispatch: (...args) => dispatch(...args)`后面被修改了，真正下发到`middleware`中的`dispatch`其实是`dispatch = compose(...chain)(store.dispatch)`处理后的，如下。
```js 
dispatch = (action) => {
    if (typeof action === 'function') {
      return action(dispatch, getState, extraArgument);
    }
    return next(action);
  };
```

### 总结
原来`redux-thunk`所具有的异步能力是对`store.dispatch`进行了一层包装，将之前仅支持的`Plain Object`拓展成还可以支持`函数类型`的，如果是普通的对象则还使用原来的`store.dispatch`，如果是函数类型则执行该函数，将包装后的`dispatch`下发到该`action`的`内部作用域`。如上就是个人最近针对redux及redux-thunk异步能力的一些理解总结，如有理解不当或书写错误的地方欢迎交流指正。
