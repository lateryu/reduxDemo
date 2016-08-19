HEAD
```
npm install
npm run server
```
### open ``http://localhost:3000/`` in browser.
=======

### reduxDemo
redux的demo
1. npm install
2. 端口3000


# 关于学习redux的笔记
## 单页面应用的痛点

* 对于复杂的单页面应用，状态（state）管理非常重要。state 可能包括：服务端的响应数据、本地对响应数据的缓存、本地创建的数据（比如，表单数据）以及一些 UI 的状态信息（比如，路由、选中的 tab、是否显示下拉列表、页码控制等等）。如果 state 变化不可预测，就会难于调试（state 不易重现，很难复现一些 bug）和不易于扩展（比如，优化更新渲染、服务端渲染、路由切换时获取数据等等）。

* Redux 就是用来确保 state 变化的可预测性，主要的约束有：
  * state 以单一对象存储在 store 对象中
  * state 只读
  * 使用纯函数 reducer 执行 state 更新
> state 为单一对象，使得 Redux 只需要维护一棵状态树，服务端很容易初始化状态，易于服务器渲染。state 只能通过 dispatch(action) 来触发更新，更新逻辑由 reducer 来执行。  


## Actions、Reducers 和 Store

### 1.store 是一个单一对象：

  store 仅是一个 state 容器。这是存储 state 并在哪里 actions 被调度及处理的地方。当您开始构建一个 Redux 应用，您要思考如何在应用中存储模块和 state。这很重要因为 Redux 建议只有一个 store，并且由于 state 共享这是之前想到的一个不错的想法。
  * 管理应用的 state
  * 通过 store.getState() 可以获取 state
  * 通过 store.dispatch(action) 来触发 state 更新
  * 通过 store.subscribe(listener) 来注册 state 变化监听器
  * 通过 createStore(reducer, [initialState]) 创建
  
> 在 Redux 应用中，只允许有一个 store 对象，可以通过 combineReducers(reducers) 来实现对 state 管理的逻辑划分（多个 reducer）。 

### 2. action
 可以理解为应用向 store 传递的数据信息（一般为用户交互信息）。在实际应用中，传递的信息可以约定一个固定的数据格式，比如: Flux Standard Action。  

  为了便于测试和易于扩展，Redux 引入了 Action Creator:
``` jsx
{
  type: 'ADD_USER',
  data: {
    name: 'Foo',
    email: 'foo@bar.com',
    password: 'Foobar123_'
  }
}

为了让操作变得更清晰和更容易复用，通常使用一个建造者模式来创建 action 对象。例如在上述情况，您可以为这个对象创建一个函数如 addUser(name, email, password)。正如您所看到的，actions 本身并不操作任何东西。action 仅仅是一个描述我们如何改变 state 的对象。

function addUser(name, email, password) {
  return {
    type: ADD_USER,
    name: name,
    email: email,
    password: password
  }
}
store.dispatch(addUser(name, email, password))
```


> dispatch(action) 是一个同步的过程：执行 reducer 更新 state -> 调用 store 的监听处理函数。如果需要在 dispatch 时执行一些异步操作（fetch action data），可以通过引入 Middleware 解决。

### 3. reducer 
  实际上就是一个函数：(previousState, action) => newState。用来执行根据指定 action 来更新 state 的逻辑。通过 combineReducers(reducers) 可以把多个 reducer 合并成一个 root reducer。
> reducer 不存储 state, reducer 函数逻辑中不应该直接改变 state 对象, 而是返回新的 state 对象（可以考虑使用 immutable-js）。

  Reducers 在 store 中通过分发处理 action 来减少这些 actions 对 state 的改变。如果我们在 store 分发一个 action 如 ADD_USER，我们可以用 reducer 将那些添加新用户的 action 入口到 state。

## 构建redux应用
如果我们想到一个 to-do 应用，我们将会需要一些基本的事项。首先一个 to-do 通常由一个列表组成。另外，这个列表包含我们可以更改的待办事项。
从一个 state 角度来看，我们这个应用的模型类似这样：
```jsx 
{
  todo: {
    items: [
      {
        message: "Finish Redux blog post...",
        completed: false
      }
    ]
  }
}
```
#### 添加Actions
```jsx
function addTodo(message) {
  return {
    type: 'ADD_TODO',
    message: message,
    completed: false
  };
}

```
**注意:** 这里的 type 字段。这个应该是个唯一的名称用于描述您的 action。通常这个类型是大写格式并用底划线作为单词分隔符。另外您将使用这个名称/标识符在 reducers 中处理具体的 actions 并改变它们的 state 。

一旦我们增加了新的待办事项，我们肯定希望能够将其标记为已完成，我们也希望能够将其删除，甚至可以清除所有待办事项。
因此让我们也为这些操作添加 actions ：
```jsx

function completeTodo(index) {
  return {
    type: 'COMPLETE_TODO',
    index: index
  };
}

function deleteTodo(index) {
  return {
    type: 'DELETE_TODO',
    index: index
  };
}

function clearTodo() {
  return {
    type: 'CLEAR_TODO'
  };
}
 ```

现在我们已经有了 actions，让我们继续构建 store。如果您还记的刚刚说的，**store 是 Redux 应用的核心** 关联所有的 state，调度 actions 和 reducers 处理。
```jsx
import { createStore } from 'redux';

var defaultState = {
  todo: {
    items: []
  }
};

function todoApp(state, action) {
}

var store = redux.createStore(todoApp, defaultState);
```

### 添加 Reducers
现在当我们有一些 actions 和 store，让我们创建第一个 reducer。如果您还记的刚刚说的，reducer 只是一个处理器您可以用它来处理 actions 和改变 state。
因此我们开始处理 ADD_TODO action 如下所示：
```jsx 
function todoApp(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return Object.assign({}, state, {
        items: items.concat([{
          message: action.message,
          completed: false
        }])
      });

    default:
      return state;
  }
}
```
> **注意:** 当我们说一个 reducer “改变” state 时，如果一个 state 需要改变，实际它所做的是创建一个 state 副本并作出改变。如果没有变化，那么返回相同的 state。但在任何情况下您都不应该直接改变 state 因为这样意味着改变 state history 

现在当我们有了第一个 action 处理器，让我们为它们增加其余的支持：

```jsx
function todoApp(state, action) {
  switch (action.type) {
    case 'ADD_TODO':
      return Object.assign({}, state, {
        todo: {
          items: items.concat([{
            message: action.message,
            completed: false
          }])
        }
      });

    case 'COMPLETE_TODO':
      var items = [].concat(state.todo.items);

      items[action.index].completed = true;

      return Object.assign({}, state, {
        todo: {
          items: items
        }
      });

    case 'DELETE_TODO':
      var items = [].concat(state.todo.items);

      items.splice(action.index, 1);

      return Object.assign({}, state, {
        todo: {
          items: items
        }
      });

    case 'CLEAR_TODO':
      return Object.assign({}, state, {
        todo: {
          items: []
        }
      });

    default:
      return state;
  }
}
```

### 把它们结合到一个 React 界面
现在我们已经有了业务逻辑，让我们写一些 UI 代码。由于大部分是常见的 React 知识并且非常类似 Flux 构建的应用, 就不再深入讲解。

```js
import React from 'react';
import ReactDOM from 'react-dom';
import { createStore } from 'redux';

var defaultState = {
  todo: {
    items: []
  }
};

// 添加我们在前面步骤中创建的 actions ...

function todoApp(state, action) {
  // 添加我们在前面步骤中的 reducer 逻辑...
}

var store = redux.createStore(todoApp, defaultState);

class AddTodoForm extends React.Component {
  state = {
    message: ''
  };

  onFormSubmit(e) {
    e.preventDefault();
    store.dispatch(addTodo(this.state.message));
    this.setState({ message: '' });
  }

  onMessageChanged(e) {
    var message = e.value.trim();
    this.setState({ message: message });
  }

  render() {
    return (
      <form onSubmit={this.onFormSubmit.bind(this)}>
        <input type="text" placeholder="Todo..." onChange={this.onMessageChanged.bind(this)} value={this.state.message} />
        <button type="submit" value="Add" />
      </form>
    );
  }
}

class TodoItem extends React.Component {
  onDeleteClick() {
    store.dispatch(deleteTodo(this.props.index));
  }

  onCompletedClick() {
    store.dispatch(completeTodo(this.props.index));
  }

  render() {
    return (
      <li>
        <a href="#" onClick={this.onDeleteClick.bind(this)}>[x]</a>
        <a href="#" onClick={this.onCompletedClick.bind(this)}>{this.props.message}</a>
      </li>
    );
  }
}

class TodoList extends React.Component {
  state = {
    items: []
  };

  componentWillMount() {
    store.subscribe(() => {
      var state = store.state();
      this.setState({
        items: state.todo.items
      });
    });
  }

  render() {
    var items = [];

    this.state.items.forEach((item, index) => {
      items.push(<TodoItem
        index={index}
        message={item.message}
        completed={item.completed}
      />);
    });

    return (
      <ol>{ items }</ol>
    );
  }
}

ReactDOM.render(
  <div>
    <h1>Todo</h1>
    <AddTodoForm /><hr />
    <TodoList />
  </div>,
  document.getElementById('app')
);
```
正如您所看到的，构建 Redux 应用 UI 部分并不难。唯一的区别就是您用 store.subscribe(listener) 来监听 state 改变，然后通过 store.getState() 检索 state。但除此之外，它非常像一个 Flux 构建的应用。

### Redux 支持 Stormpath React SDK

因为有相当多需要增加 Redux 支持 `Stormpath React SDK` 需求，我们说干就干。如果您希望为 Redux 配置这个 SDK，简单的配置这个 ``Stormpath React SDK dispatcher`` 选项并设置 type 为 redux 指向 store 到您的 Redux store 如下所示：
```jsx
function myApp(state, action) {
  return state;
}

ReactStormpath.init({
  dispatcher: {
    type: 'redux',
    store: createStore(myApp)
  }
});

```


一旦完成这些您可以拦截并处理一切 `Stormpath React SDK` 的 actions 调度。例如，您希望用用户数据来丰富 state，然后简单处理这个 USER_SET action 并将用户数据添加到 state。
```jsx

function myApp(state, action) {
  switch (action.type) {
    case 'USER_SET':
      return Object.assign({}, state, {
        user: action.data
      });

    default:
      return state;
  }
}
```


## 连接React和Redux

要在React的项目中使用Redux，比较好的方式是借助react-redux这个库来做连接，这里的意思是，并不是没有react-redux，这两个库就不弄一起用了，而是说react-redux提供了一些封装，一种更科学的代码组织方式，让我们更舒服地在React的代码中使用Redux。
react-redux提供两个关键模块：Provider和connect。

### Provider

Provider这个模块是作为整个App的容器，在你原有的App Container的基础上再包上一层，它的工作很简单，就是接受Redux的store作为props，并将其声明为context的属性之一，子组件可以在声明了contextTypes之后可以方便的通过this.context.store访问到store。不过我们的组件通常不需要这么做，将store放在context里，是为了给下面的connect用的。

这个是Provider的使用示例：
```jsx
// config app root
const history = createHistory()
const root = (
  <Provider store={store} key="provider">
    <Router history={history} routes={routes} />
  </Provider>
)

// render
ReactDOM.render(
  root,
  document.getElementById('root')
)
```

### connect
这个模块是算是真正意义上连接了Redux和React，正好它的名字也叫connect。

先考虑Redux是怎么运作的：首先store中维护了一个state，我们dispatch一个action，接下来reducer根据这个action更新state。

映射到我们的React应用中，store中维护的state就是我们的app state，一个React组件作为View层，做两件事：render和响应用户操作。于是connect就是将store中的必要数据作为props传递给React组件来render，并包装action creator用于在响应用户操作时dispatch一个action。

好了，详细看看connect这个模块做了什么。先从它的使用来说，它的API如下：

```
connect([mapStateToProps], [mapDispatchToProps], [mergeProps], [options])
```
> mapStateToProps是一个函数，返回值表示的是需要merge进props的state。默认值为() => ({})，即什么都不传。
```
(state, props) => ({ }) // 通常会省略第二个参数
```

mapDispatchToProps是可以是一个函数，返回值表示的是需要merge仅props的actionCreators，这里的actionCreator应该是已经被包装了dispatch了的，推荐使用redux的bindActionCreators函数。

```jsx
(dispatch, props) => ({ // 通常会省略第二个参数
 ...bindActionCreators({
   ...ResourceActions
 }, dispatch)
})
```

更方便的是可以直接接受一个对象，此时connect函数内部会将其转变为函数，这个函数和上面那个例子是一模一样的。

mergeProps用于自定义merge流程，下面这个是默认流程，parentProps值的就是组件自身的props，可以发现如果组件的props上出现同名，会被覆盖。
```
(stateProps, dispatchProps, parentProps) => ({
  ...parentProps,
  ...stateProps,
  ...dispatchProps
})
```

options共有两个开关：pure代表是否打开优化，详细内容下面会提，默认为true，withRef用来给包装在里面的组件一个ref，可以通过getWrappedInstance方法来获取这个ref，默认为false。

connect返回一个函数，它接受一个React组件的构造函数作为连接对象，最终返回连接好的组件构造函数。

然后几个问题：

* React组件如何响应store的变化？
* 为什么connect选择性的merge一些props，而不是直接将整个state传入？
* pure优化的是什么？

我们把connect返回的函数叫做Connector，它返回的是内部的一个叫Connect的组件，它在包装原有组件的基础上，还在内部监听了Redux的store的变化，为了让被它包装的组件可以响应store的变化:

```jsx
trySubscribe() {
  if (shouldSubscribe && !this.unsubscribe) {
    this.unsubscribe = this.store.subscribe(::this.handleChange)
    this.handleChange()
  }
}

handleChange () {
  this.setState({
    storeState: this.store.getState()
  })
}
```

但是通常，我们connect的是某个Container组件，它并不承载所有App state，然而我们的handler是响应所有state变化的，于是我们需要优化的是：当storeState变化的时候，仅在我们真正依赖那部分state变化时，才重新render相应的React组件，那么什么是我们真正依赖的部分？就是通过mapStateToProps和mapDispatchToProps得到的。

具体优化的方式就是在shouldComponentUpdate中做检查，如果只有在组件自身的props改变，或者mapStateToProps的结果改变，或者是mapDispatchToProps的结果改变时shouldComponentUpdate才会返回true，检查的方式是进行shallowEqual的比较。

所以对于某个reducer来说：
```
export default (state = {}, action) => {
  return { ...state } // 返回的是一个新的对象，可能会使组件reRender
  // return state // 可能不会使得组件reRender
}
```

另外在connect的时候，要谨慎map真正需要的state或者actionCreators到props中，以避免不必要的性能损失。

最后，根据connect的API我们发现可以使用ES7 decorator功能来配合React ES6的写法：

```
@connect(
  state => ({
    user: state.user,
    resource: state.resource
  }),
  dispatch => ({
    ...bindActionCreators({
      loadResource: ResourceActions.load
    }, dispatch)
  })
)
export default class Main extends Component {

}
```



# 对rudex的深入理解 20160810




## 简单入门
```js
// 首先定义一个改变数据的plain函数，成为reducer
function count (state, action) {
    var defaultState = {
        year: 2015,
      };
    state = state || defaultState;
    switch (action.type) {
        case 'add':
            return {
                year: state.year + 1
            };
        case 'sub':
            return {
                year: state.year - 1
            }
        default :
            return state;
    }
}

// store的创建
var createStore = require('redux').createStore;
var store = createStore(count);

// store里面的数据发生改变时，触发的回调函数
store.subscribe(function () {
      console.log('the year is: ', store.getState().year);
});

// action: 触发state改变的唯一方法(按照redux的设计思路)
var action1 = { type: 'add' };
var action2 = { type: 'add' };
var action3 = { type: 'sub' };

// 改变store里面的方法
store.dispatch(action1); // 'the year is: 2016
store.dispatch(action2); // 'the year is: 2017
store.dispatch(action3); // 'the year is: 2016
```

对 createStore 的理解

1. createStore的返回的内容

```js 
export default function createStore(reducer, initialState) {
    ...
    return {
        dispatch,
        subscribe,
        getState,
        replaceReducer
    }
}
```
每个属性的含义是:
  * dispatch: 用于action的分发，改变store里面的state
  * subscribe: 注册listener，store里面state发生改变后，执行该listener
  * getState: 读取store里面的state
  * replaceReducer: 替换reducer，改变state修改的逻辑

2. 关键代码解析

```js
export default function createStore(reducer, initialState) {
    // 这些都是闭包变量
    var currentReducer = reducer
    var currentState = initialState
    var listeners = []
    var isDispatching = false;

    // 返回当前的state
    function getState() {
        return currentState
    }

    // 注册listener，同时返回一个取消事件注册的方法
    function subscribe(listener) {
        listeners.push(listener)
        var isSubscribed = true

        return function unsubscribe() {
            if (!isSubscribed) {
                return
            }

            isSubscribed = false
            var index = listeners.indexOf(listener)
            listeners.splice(index, 1)
        }
    }

    // 通过action该改变state，然后执行subscribe注册的方法
    function dispatch(action) {
        try {
          isDispatching = true
              currentState = currentReducer(currentState, action)
        } finally {
              isDispatching = false
        }
        listeners.slice().forEach(listener => listener())
        return action
    }

    // 替换reducer，修改state变化的逻辑
    function replaceReducer(nextReducer) {
           currentReducer = nextReducer
           dispatch({ type: ActionTypes.INIT })
       }

       // 初始化时，执行内部一个dispatch，得到初始state
       dispatch({ type: ActionTypes.INIT })
}
```

3. 随着应用越来越大，一方面，不能把所有的数据都放到一个reducer里面，另一方面，为每个reducer创建一个store，后续store的维护就显得比较麻烦。如何将二者统一起来呢？


  通过combineReducers将多个reducer合并成一个rootReducer:
```js
// 创建两个reducer: count year
function count (state, action) {
  state = state || {count: 1}
  switch (action.type) {
    default:
      return state;
  }
}
function year (state, action) {
  state = state || {year: 2015}
  switch (action.type) {
    default:
      return state;
  }
}

// 将多个reducer合并成一个
var combineReducers = require('./').combineReducers;
var rootReducer = combineReducers({
  count: count,
  year: year,
});

// 创建store，跟2.1没有任何区别
var createStore = require('./').createStore;
var store = createStore(rootReducer);

var util = require('util');
console.log(util.inspect(store));
//输出的结果，跟2.1的store在结构上不存在区别
// { dispatch: [Function: dispatch],
//   subscribe: [Function: subscribe],
//   getState: [Function: getState],
//   replaceReducer: [Function: replaceReducer]
// }
```

3.1  源码解析combineReducers
  ```jsx
// 高阶函数，最后返回一个reducer
export default function combineReducers(reducers) {
    // 提出不合法的reducers, finalReducers就是一个闭包变量
    var finalReducers = pick(reducers, (val) => typeof val === 'function')
    // 将各个reducer的初始state均设置为undefined
    var defaultState = mapValues(finalReducers, () => undefined)

    // 一个总reducer，内部包含子reducer
    return function combination(state = defaultState, action) {
        var finalState = mapValues(finalReducers, (reducer, key) => {
            var previousStateForKey = state[key]
            var nextStateForKey = reducer(previousStateForKey, action)
            hasChanged = hasChanged || nextStateForKey !== previousStateForKey
            return nextStateForKey
        }
    }

    return hasChanged ? finalState : state

}

  ```
4. 自动实现dispatch

  4.1 demo介绍
  在2.1中，要执行state的改变，需要手动dispatch:
  ```js
var action = { type: '***', payload: '***'};
dispatch(action);
  ```
手动dispatch就显得啰嗦了，那么如何自动完成呢?
```js
var bindActionCreators = require('redux').bindActionCreators;
// 可以在具体的应用框架隐式进行该过程(例如react-redux的connect组件中)
bindActionCreators(action)
```

4.2 源码解析
```js
// 隐式实现dispatch
function bindActionCreator(actionCreator, dispatch) {
  return (...args) => dispatch(actionCreator(...args))
}

export default function bindActionCreators(actionCreators, dispatch) {
    if (typeof actionCreators === 'function') {
        return bindActionCreator(actionCreators, dispatch)
    }
    return mapValues(actionCreators, actionCreator =>
        bindAQctionCreator(actionCreator, dispatch)
    )
}
```
5. 支持插件 - 对dispatch的改造

5.1 插件使用demo
一个action可以是同步的，也可能是异步的，这是两种不同的情况， dispatch执行的时机是不一样的:

```js
// 同步的action creator, store可以默认实现dispatch
function add() {
    return { tyle: 'add' }
}
dispatch(add());

// 异步的action creator，因为异步完成的时间不确定，只能手工dispatch
function fetchDataAsync() {
    return function (dispatch) {
        requst(url).end(function (err, res) {
            if (err) return dispatch({ type: 'SET_ERR', payload: err});
            if (res.status === 'success') {
                dispatch({ type: 'FETCH_SUCCESS', payload: res.data });
            }
        })
    }
}
```

下面的问题就变成了，如何根据实际情况实现不同的dispatch方法，也即是根据需要实现不同的moddleware:

```js
// 普通的dispatch创建方法
var store = createStore(reducer, initialState);
console.log(store.dispatch);

// 定制化的dispatch
var applyMiddleware = require('redux').applyMiddleware;
// 实现action异步的middleware
var thunk = requre('redux-thunk');
var store = applyMiddleware([thunk])(createStore);
// 经过处理的dispatch方法
console.log(store.dispatch);
```

5.2 源码解析

```js
// next: 其实就是createStore
export default function applyMiddleware(...middlewares) {
  return (next) => (reducer, initialState) => {
    var store = next(reducer, initialState)
    var dispatch = store.dispatch
    var chain = []

    var middlewareAPI = {
      getState: store.getState,
      dispatch: (action) => dispatch(action)
    }
    chain = middlewares.map(middleware => middleware(middlewareAPI))
    dispatch = compose(...chain)(store.dispatch)

    return {
      ...store,
      dispatch // 实现新的dispatch方法
    }
  }
}
// 再看看redux-thunk的实现, next就是store里面的上一个dispatch
function thunkMiddleware({ dispatch, getState }) {
    return function(next) {
        return function(action) {
            typeof action === 'function' ?
            action(dispatch, getState) :
            next(action);
        }
    }
  return next => action =>
    typeof action === 'function' ?
      action(dispatch, getState) :
      next(action);
}
```

6. 与react框架的结合
6.1 基本使用
目前已经有现成的工具react-redux来实现二者的结合:

```js
ar rootReducers = combineReducers(reducers);
var store = createStore(rootReducers);
var Provider = require('react-redux').Provider;
// App 为上层的Component
class App extend React.Component{
    render() {
        return (
            <Provier store={store}>
                <Container />
            </Provider>
        );
    }
}

// Container作用: 1. 获取store中的数据; 2.将dispatch与actionCreator结合起来
var connect = require('react-redux').connect;
var actionCreators = require('...');
// MyComponent是与redux无关的组件
var MyComponent = require('...');

function select(state) {
    return {
        count: state.count
    }
}
export default connect(select, actionCreators)(MyComponent)

```


6.2 Provider -- 提供store
React通过Context属性，可以将属性(props)直接给子孙component，无须通过props层层传递, Provider仅仅起到获得store，然后将其传递给子孙元素而已:

```js
export default class Provider extends Component {
  getChildContext() { // getChildContext: 将store传递给子孙component
    return { store: this.store }
  }

  constructor(props, context) {
    super(props, context)
    this.store = props.store
  }

  componentWillReceiveProps(nextProps) {
    const { store } = this
    const { store: nextStore } = nextProps

    if (store !== nextStore) {
      warnAboutReceivingStore()
    }
  }

  render() {
    let { children } = this.props
    return Children.only(children)
  }
}
```


6.3 connect -- 获得store及dispatch(actionCreator)(connect方法将我们需要的state中的数据和actions中的方法绑定到props上)

1. 第一个参数，必须是function，作用是绑定state的指定值到props上面。这里绑定的是counter
2. 第二个参数，可以是function，也可以是对象，作用是绑定action创建函数到props上。
3. 返回值，是绑定后的组件

connect是一个高阶函数，首先传入mapStateToProps、mapDispatchToProps，然后返回一个生产Component的函数(wrapWithConnect)，然后再将真正的Component作为参数传入wrapWithConnect(MyComponent)，这样就生产出一个经过包裹的Connect组件，该组件具有如下特点:

* 通过this.context获取祖先Component的store
* props包括stateProps、dispatchProps、parentProps,合并在一起得到
* nextState，作为props传给真正的Component
* componentDidMount时，添加事件this.store.subscribe(this.handleChange)，实现页面交互
shouldComponentUpdate时判断是否有避免进行渲染，提升页面性能，并得到nextState
componentWillUnmount时移除注册的事件this.handleChange
在非生产环境下，带有热重载功能

```jsx
import { bindActionCreators } from 'redux'
import { connect } from 'react-redux'
import Counter from '../components/Counter'
import * as CounterActions from '../actions/counter'

//将state.counter绑定到props的counter
function mapStateToProps(state) {
  return {
    counter: state.counter
  }
}
//将action的所有方法绑定到props上
function mapDispatchToProps(dispatch) {
  return bindActionCreators(CounterActions, dispatch)
}

//通过react-redux提供的connect方法将我们需要的state中的数据和actions中的方法绑定到props上
export default connect(mapStateToProps, mapDispatchToProps)(Counter)
```



