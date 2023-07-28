---
title: Redux
date: 2023-07-27 17:48:19
tags:
 - Redux
categories:
 - Redux
---

# Redux

github 
```
https://github.com/reduxjs/redux
```
v5 版本


核心 API

## createStore

已经准备被 废弃，使用 @reduxjs/toolkit '包**中的' configureStore 方法
或者使用 legacy_createStore 方法

返回的 store 属性
方法内部的很多容错处理都是依赖了 isDispatching 属性
这样可以避免正在 dispatch 的时候，去进行更改出现错误

```JavaScript

const store = {
  dispatch,
  subscribe,
  // 获取当前的 state
  getState,
  replaceReducer,
  // $$observable 是一个通过 Symbol 直接创建的唯一性 键值对
  [$$observable]: observable
}

```

### dispatch

很简单主要就是两步
调用 reducer 更新 state
调用 listeners 监听

```JavaScript

function dispatch(action: A) {
  // 如果 isDispatching 直接抛出异常
  if (isDispatching) {
    throw new Error()
  }

  try {
    isDispatching = true
    // currentReducer 是创建时的 reducer 方法
    // 或者时 replaceReducer 之后的 reducer 方法
    currentState = currentReducer(currentState, action)
  } finally {
    isDispatching = false
  }

  // 调用 listeners 监听
  const listeners = (currentListeners = nextListeners)
  listeners.forEach(listener => {
    listener()
  })
  return action
}

```

### subscribe

注入 listeners 监听方法
返回 unlistener 方法

其中 ensureCanMutateNextListeners 方法：
  保证后续对 next 的操作不会影响到 current

```JavaScript

function subscribe(listener: () => void) {

  let isSubscribed = true

  // 这个可以将监听器赋值，确保了可以 更改 nextListeners。
  // 会在 dispatch 之后将监听器进行更新。
  // 所以在更新之后，如果需要添加监听，需要重新将 nextListeners
  // 创建一份
  ensureCanMutateNextListeners()
  // 这里是 add listeners 
  const listenerId = listenerIdCounter++
  nextListeners.set(listenerId, listener)

  return function unsubscribe() {
    isSubscribed = false

    // 这个也是同理，保证删除的 next 不会影响到 current
    ensureCanMutateNextListeners()
    nextListeners.delete(listenerId)
    // 这里的赋值作用，不确定有无作用。
    // 因为在下一次 dispatch 会更新，不知道这个 
    // 设置为 null 的作用是为了避免再次执行到以前的监听逻辑吗？
    currentListeners = null
  }
}

```


### getState

就是直接返回 currentState

```JavaScript

function getState(): S {
  if (isDispatching) {
    throw new Error()
  }

  return currentState as S
}

```


### replaceReducer

用于替换 reducer
替换之后调用了一次 dispatch 作用应该是为了更新 state 的值

```JavaScript

function replaceReducer(nextReducer: Reducer<S, A>): void {
    currentReducer = nextReducer as unknown as Reducer<S, A, PreloadedState>
    dispatch({ type: ActionTypes.REPLACE } as A)
  }

```


### observable

观察者模式

内部本质还是 subscribe
但是会增加了参数，会调用 next 方法处理 state 数据
可能和 rxjs 异步等有关联？？

```JavaScript


function observable() {
  const outerSubscribe = subscribe
  return {
    // subscribe 方法，传递 observer 参数 存在 next 属性进行调用
    subscribe(observer: unknown) {
      function observeState() {
        const observerAsObserver = observer as Observer<S>
        if (observerAsObserver.next) {
          // 调用 next 参数为 state 的数据
          observerAsObserver.next(getState())
        }
      }

      observeState()
      // 加入 listeners 的监听数组
      const unsubscribe = outerSubscribe(observeState)
      return { unsubscribe }
    },

    [$$observable]() {
      return this
    }
  }
}

```


## combineReducers

连接 reducers 方法，
对于多个 reducers 我们正常可能需要使用用两个去进行存储
此时这里就用这中方法将两个连接起来


```JavaScript

// reducer1
function todos(state = [], action) {
  switch (action.type) {
  case 'ADD_TODO':
    return state.concat([action.text])
  default:
    return state
  }
}

// reducer2
function counter(state = 0, action) {
  switch (action.type) {
  case 'INCREMENT':
    return state + 1
  case 'DECREMENT':
    return state - 1
  default:
    return state
  }
}

// 连接起来
const reducers = combineReducers({todos, counter});
cosnt store = createStore(reducers);
// 这里的 键 是取决于 combineReducers 传递的对象的键
console.log(store.getState()); // { todos: [], counter: 0 }

```


核心代码
```JavaScript

export default function combineReducers(reducers: {
  [key: string]: Reducer<any, any, any>
}) {
  const reducerKeys = Object.keys(reducers)

  return function combination(
    state: StateFromReducersMapObject<typeof reducers> = {},
    action: Action
  ) {
    // 判断最后是否有更新
    let hasChanged = false
    // 存储的 nextState
    const nextState: StateFromReducersMapObject<typeof reducers> = {}
    // 循环调用 reducers
    for (let i = 0; i < reducerKeys.length; i++) {
      const key = reducerKeys[i]
      // 取出 reducer
      const reducer = reducers[key]
      const previousStateForKey = state[key]
      // 执行 reducer
      const nextStateForKey = reducer(previousStateForKey, action)
      
      // 将数据放进去 nextState key 作为键
      nextState[key] = nextStateForKey
      // 代表判断是否两次的值是否发生了变化。
      hasChanged = hasChanged || nextStateForKey !== previousStateForKey
    }
    hasChanged =
      hasChanged || reducerKeys.length !== Object.keys(state).length
    // 判断是否发生了 change
    return hasChanged ? nextState : state
  }
}

```


## bindActionCreators

连接 action 的方法
同 连接 reducer 一个意思

eg：
对于多个事件工厂 action1 action2
action1() action2()
原本的调用思路 dispatch(action1()) / dispatch(action2())
通过 newDispatch = bindActionCreators({action1, action2}, dispatch)
可以改变调用方式 newDispatch.action1() ...


```JavaScript

// bindActionCreator 核心代码
function bindActionCreator<A extends Action>(
  actionCreator: ActionCreator<A>,
  dispatch: Dispatch<A>
) {
  return function (this: any, ...args: any[]) {
    return dispatch(actionCreator.apply(this, args))
  }
}

// bindActionCreators 核心代码
export default function bindActionCreators(
  actionCreators: ActionCreator<any> | ActionCreatorsMapObject,
  dispatch: Dispatch
) {
  // 只有一个的情况
  if (typeof actionCreators === 'function') {
    return bindActionCreator(actionCreators, dispatch)
  }

  const boundActionCreators: ActionCreatorsMapObject = {}
  for (const key in actionCreators) {
    // 取出每次的 action工厂
    const actionCreator = actionCreators[key]
    if (typeof actionCreator === 'function') {
      // 放进去一个统一的工厂中
      boundActionCreators[key] = bindActionCreator(actionCreator, dispatch)
    }
  }
  // 返回
  return boundActionCreators
}

```


## applyMiddleware

applyMiddleware
返回的dispatch将是执行过了中间件的 dispatch
内部是一个类似于的洋葱模型，会先执行外层到内层的数据
然后在 dispatch 的时候由从内层到外层

eg: 代码
```JavaScript

const middle1 = (store) => {
  console.log(store.getState());
  try {
    console.log(stroe.dispatch());
  } catch (e) {
    console.log('这个时候不能调用 dispatch，因为是在中间件构造的时候');
  }

  // 返回一个 用 disptach 参数的方法
  // 会在 applyMiddleware 就已经开始执行了
  // 作用可能接近于初始化，同时包含了 dispatch 方法
  // 这个还包含了闭包的考虑
  return (dispatch) => {
    console.log('封装的外层 applyMiddleware 此时就已经开始被执行');
    // 这个 return 的方法，将被当作下一个的 dispatch
    // 这个是一个由内部到外部的执行顺序。
    // 同时每次执行都会触发 但是是数据更新之前
    return (action) => {
      console.log('此时才开始执行 dispatch，但是这个 dispatch 其实是后面的传递过来的');
      return dispatch(action)
    }
  }
}
const middle2 = (store) => {
  return (dispatch) => {
    console.log('同上，在调用 applyMiddleware 此时就已经开始被执行');
    return (action) => {
      console.log('此时才开始执行 dispatch');
      return dispatch(action)
    }
  }
}

const store = createStore(reducer, applyMiddleware(middle1, middle2))
console.log(store.getState());
store.dispatch({type: 'INCREMENT'})

/*
输出结果： state = { todos: [], counter: 0 }
{ todos: [], counter: 0 }
这个时候不能调用 dispatch，因为是在中间件构造的时候
同上，在调用 applyMiddleware 此时就已经开始被执行
封装的外层 applyMiddleware 此时就已经开始被执行
{ todos: [], counter: 0 }
此时才开始执行 dispatch，但是这个 dispatch 其实是后面的传递过来的 
此时才开始执行 dispatch

*/

```

```JavaScript

export default function applyMiddleware(
  ...middlewares: Middleware[]
): StoreEnhancer<any> {
  return createStore => (reducer, preloadedState) => {
    const store = createStore(reducer, preloadedState)
    // 给一个默认的 dispatch，表示的是，不能在构造中间件的时候调度 dispatch
    // 从代码上看来说，是不能在中间件中调用 dispatch 
    let dispatch: Dispatch = () => {
      // 不允许在构造中间件时进行调度。其他中间件不会应用于此分派。
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }

    const middlewareAPI: MiddlewareAPI = {
      getState: store.getState,
      dispatch: (action, ...args) => dispatch(action, ...args)
    }
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // compose 主要是一个实现了洋葱模型的方法，可以将参数传递进去之后，然后返回一个方法，
    // 这个方法的参数将作为第一个执行的参数，然后后续的参数将依赖前面执行的方法的返回值
    dispatch = compose<typeof dispatch>(...chain)(store.dispatch)

    return {
      ...store,
      dispatch
    }
  }
}


```



## compose

一个实现了洋葱模型的函数

里面看着可能比较乱
首先 外层的参数是第一个参数，当然我们内层也可以不需要返回方法
只是因为为了了解 redux 的 middle 内部是怎么执行的逻辑顺序，
所以 compose() 返回后调用的参数 是一个方法，代替的是 dispatch

```JavaScript

const fn1 = (func1) => {
  console.log('fn1', func1);
  return (arg1) => {
    console.log('fn1 return', arg1);
    return func1(arg1 + 1);
  };
}
const fn2 = (func2) => {
  console.log('fn2', func2);
  return (arg2) => {
    console.log('fn2 return', arg2);
    return func2(arg2 + 1);
  };
}

compose(fn1, fn2)(() => 1)(4)

/*

fn2 [Function (anonymous)]
fn1 [Function (anonymous)]
fn1 return 4
fn2 return 5

*/

```

```javaScript

export default function compose(...funcs: Function[]) {
  // 重点!!!
  return funcs.reduce(
    (a, b) =>
      (...args: any) =>
        a(b(...args))
  )
  // 详细描述
  // 不传递参数，表示 第一个是从下标0开始
  // 将一个没有执行的方法整体放成 preFn
  // eg:  fn1, fn2, fn3
  // preFn = fn1, curFn = fn2
  // preFn(curFn(...)) = fn1(fn2(...))
  // preFn = fn1(fn2(...)), curFn = fn3
  // preFn(curFn(...)) = fn1(fn2(fn3(...))); 
  /**
   * 详细逻辑
   * fn1, fn2, fn3
   * 第一次
   * preFn = fn1, curFn = fn2
   * reutrn (...a) => {
   *   return fn1(fn2(...a));
   * } as fn1fn2
   * 第二次
   * preFn = fn1fn2, curFn = fn3
   * return (...a) => {
   *   return (fn3(...a) as args) => {
   *     return fn1(fn2(...args));
   *   }
   * }
   */
  return funcs.reduce(
    (preFn, curFn) => {
      // 将这个方法直接返回，就会变成下一个的 preFn 
      return (...args: any[]) => {
        // 执行的是 preFn(curFn())
        return preFn(curFn(...args))
      }
    }
  )
}


```