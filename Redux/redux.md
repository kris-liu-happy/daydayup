## 1.什么情况下会使用redux

- **某个组件的信息， 需要共享**
- **某个状态，需要在任何地方都能拿到**
- **某个组件，需要改变全局的状态**
- **某个组件，需要改变另一个组件的状态**



## 2.设计思想

​	引用阮一峰老师的设计思想

​	（1）Web 应用是一个状态机，视图与状态是一一对应的。

​	（2）所有的状态，保存在一个对象里面。



## 3.基本概念和 API

#### 3.1   store

Store 是储存数据的地方，可以把它看成一个容器。

store 是通过createStore 来生成的

```js
import { createStore } from 'redux';

// createStore 接受三个参数，reducer initState applyMiddleware
const store = createStore(Fn);
```



#### 3.2 State

Store 的所有数据都放在state 里面的， createStore 的第二个参数(initState)传入初始值

当前的state 可以通过 store.getState()拿到

```js
import { createStore } from 'redux';
const initState = {a:1,b:2}
const store = createStore(Fn, initState);

const state = store.getState();
console.log(state)
// {a:1,b:2}
```



#### 3.3 Action

Action是一个对象，type 属性是必须的

```js
eg. 

value : 0

按钮： +      -
  
type   -: mimus   +:plus
 
const action = {type, payload}

现在有2个按钮   一个减 一个加。  我们点击不同的按钮时，就是用type来区分


```



Action 描述当前发生的事情。改变 State 的唯一办法，就是使用 Action，它会运送数据到 Store。





#### 3.4   Action Creator

View 要发送多少种消息，就会有多少种 Action。如果都手写，会很麻烦。可以定义一个函数来生成 Action，这个函数就叫 Action Creator。



```

const type = '+1';

function addTodo(payload) {
  return {
    type,
    payload
  }
}

const action = addTodo('Learn Redux');


// addTodo  这个函数就是 Action Creator
```





#### 3.5 Store.dispatch

Action 创建好了，怎么把它放在store 里面来执行，就是通过Store.dispatch来派送的

```js
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```





#### 3.6 Reducer

 Store.dispatch 送来了action ，Reducer 的作用来了，就是来接收action 和 当前state，然后执行，来返回一个新的state

```js
const reducer = function (state, action) {
  // ...
  return new_state;
};
```



reducer  是一个纯函数，同样的输入，就用同样的输出

```js
function reducer(state, action) {
  return Object.assign({}, state, { thingToChange });
  // 或者
  return { ...state, ...newState };
}
```





#### 3.7 Store.subscribe

Store.subscribe 是对state 进行的一个监听，state变动了，就会触发这个函数

```js

import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);

// 解除监听(调用这个函数)
const unSubscribe = store.subscribe(() => console.log('subscribe'))

unSubscribe()
```







## 4.Store 的简单实现

```js
redux 提供了3个方法
1.Store.dispatch()
2.Store.subscribe()
3.Store.getState()


// createStore 接受三个参数
import { createStore } from 'redux';
let { subscribe, dispatch, getState } = createStore(reducer， initState, applyMiddleware);


// store 的简单实现
const createStore = (reducer， initState) => {
  let state = initState
  let listeners = [];
  
  const getState = () => state;
  
  const dispatch = (action) => {
		state = reducer(state, action);
    listeners.forEach(listener => listener());
  }
  
  const subscribe = (listener) => {
    listeners.push(listener);
    return () => {
      listeners = listeners.filter(l => l !== listener);
    }
  };
  
  return { getState, dispatch, subscribe };
}


```





## 5.applyMiddleware(中间件)

描述：对store.dispatch() 方法进行的改造，在发出action和执行reducer 之间添加了新的功能

中间件：redux-promise, redux-thunk, redux-logger

详解：[applyMiddleware(中间件)](https://www.ruanyifeng.com/blog/2016/09/redux_tutorial_part_two_async_operations.html)





## 6.redux-promise

```js
// 下面就是redux-promise 源码实现

1.判断是不是 Promise
2.是,执行了这个函数，然后在dispatch(action)
3.不是,就直接dispatch(action)

import isPromise from 'is-promise';
import { isFSA } from 'flux-standard-action';

export default function promiseMiddleware({ dispatch }) {
  return next => action => {
    if (!isFSA(action)) {
      return isPromise(action) ? action.then(dispatch) : next(action);
    }

    return isPromise(action.payload)
      ? action.payload
          .then(result => dispatch({ ...action, payload: result }))
          .catch(error => {
            dispatch({ ...action, payload: error, error: true });
            return Promise.reject(error);
          })
      : next(action);
  };
}

```

