---
title: webpack-4-搭建-React-架构：引入Redux（三）
---
####动机
随着 JavaScript 单页应用开发越来越复杂，Javascript 需要管理很多 state（状态）。这些状态可能包括服务器响应、缓存数据、本地生产尚未持久化到服务器的数据，也包括 UI 状态，如激活的路由，被选中的标签，是否显示加载动效或者分页器等。

管理不断变化的 state 很困难。如果一个 model 的变化的会引起另一个 model 的变化，那么当 view 变化时，就可能引起对应 model 以及其他 model 的变化，依次地，可能引起另一个 view 的变化。这时候的 state 就变得不受控制，不可预测，就会出现很多 bug，调试起来就很找到问题所在。

所以 Redux 就是用来解决复杂 state 多变，不可预测等问题。

####什么是 Redux

Redux 是 JavaScript 应用程序的可预测的状态容器。它将所有的状态集中到一起管理。页面不直接修改state，而是发一个 action 给到 reducer 来操作 state，并将新的 state 通知给 store，store 来进行页面的状态更新。

#####Redux的三个基本原则：

* 单一数据源（整个应用的 state 被存储在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中）
* State 是只读的（唯一改变 state 的方法就是触发 action，action 就是一个普通的 JavaScript 对象）
* 使用纯函数来执行（通过编写 reducer，来描述 action 如何改变 state tree，reducer 就是一个纯函数，接收 state 和 action 两个参数，最后返回新的 state）

####创建一个 store
* 首先安装 redux，redux 是独立于 UI 框架的工具，不依赖任何前端框架
```
npm install redux --save-dev
```
* 在 components 目录下创建一个 TodoList 目录，在此目录下创建一个 store.js，写入如下代码
```
//components/TodoList/store.js
import { createStore } from 'redux'
const initialState = {
    count: 0
}
const ADD = 'ADD'
function reducer(state = initialState, action){
    console.log(state, action)
    switch(action.type){
        case ADD:
            return { count: state.count + 1 }
        default:
            return state
    }
}
const store = createStore(reducer, initialState)
console.log(store)
export default store
```
* 将 store.js 引入 Layout.js，来看一下 store 是什么样子的
```
//components/Layout/Layout.js
···
+ import store from '../../components/TodoList/store.js'
···
```
* 运行代码，打开控制台可以看到打印出来的 store
![](https://upload-images.jianshu.io/upload_images/5541401-230a3a67a3eee35e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到 store 对象，里面有 dispatch，getState，subscribe 等方法，上面的 state 是初始化 store 的时候默认会执行一遍 reducer 方法，传入 state，action 是传入的 redux INIT 事件，getState 方法获取最新的完整的 state 对象。
* 接下来更新 store 里面的数据，通过 store.dispatch 方法
```
//components/TodoList/store.js
···
+ store.dispatch({ type: ADD })
+ console.log('dispatch 之后改变的state', store.getState())
export default store
```
可以看到 state 改变了，reducer 里面 return 的一定要是一个新的对象，对比前后不同的 state 来做数据更新
![](https://upload-images.jianshu.io/upload_images/5541401-7e72b9433fdb3ecc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
store 还有个方法 subscribe，这个接受一个回调函数，在 state 变化的时候执行，
```
//components/TodoList/store.js
···
const store = createStore(reducer, initialState)

//console.log(store)
//console.log(store.getState())
store.dispatch({ type: ADD })
//console.log('dispatch 之后改变的state', store.getState())
store.subscribe(() => {
    console.log('执行subscribe回调', store.getState())
})
store.dispatch({ type: ADD })

export default store
```
![](https://upload-images.jianshu.io/upload_images/5541401-119c0110f85bc4cc.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到在第二次 dispatch 的时候，subscribe 执行了。

####Redux 中的 reducer
reducer 是一个纯粹的 JavaScript 方法，接收 state 和 action 作为参数，根据 action 对象里的 type 属性来操作 state，返回一个新的 state 对象（return { ...state, count: action.count + 1 }）。reducer 应该是一个纯粹的方法，不应该有副作用，不应该依赖 reducer 函数外部的变量来更新 state，而是应该将变量放到 state 对象中。
根据不同的模块，可以写多个 reducer，然后通过 combineReducers 进行合并。

现在再写一个 user 的 reducer，然后与 counter 的 reducer 合并，代码如下
```
//components/TodoList/store.js
import { createStore, combineReducers } from 'redux'

const initialState = {
    count: 0
}

//user模块
const userInitialState = {
    username: 'john',
    age: 28,
    address: 'shanghai'
}

const ADD = 'ADD'
function counterReducer(state = initialState, action){
    switch(action.type){
        case ADD:
            return { count: state.count + 1 }
        default:
            return state
    }
}

const UPDATE_USERNAME = 'UPDATE_USERNAME'
function userReducer(state = userInitialState, action){
    switch(action.type){
        case UPDATE_USERNAME:
            return {
                ...state,
                username: action.newName
            }
        default:
            return state
    }
}
//combineReducers来合并counter 和 user
const allReducers = combineReducers({
    counter: counterReducer,
    user: userReducer
})
//将合并的reducer传给store，传入的初始状态也要合并
const store = createStore(allReducers, {
    counter: initialState,
    user: userInitialState
})

store.dispatch({ type: ADD })
store.dispatch({ type: UPDATE_USERNAME, newName: 'lilei' })
console.log('合并的state', store.getState())

export default store
```
运行代码，可以看到，state 被合并在一起了
![](https://upload-images.jianshu.io/upload_images/5541401-5018e7c683816a6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

####Redux 中的 action
action 就是一个普通对象，对象有个 type 属性，来表明这一次的操作是什么，还有其他的普通属性。可以使用方法来创建一个 action，返回一个对象。
```
import { createStore, combineReducers } from 'redux'

const initialState = {
    count: 0
}

//user模块
const userInitialState = {
    username: 'john',
    age: 28,
    address: 'shanghai'
}

const ADD = 'ADD'
function counterReducer(state = initialState, action){
    // console.log(state, action)
    switch(action.type){
        case ADD:
            return { count: action.num + 1 }  //用传进来的action.num来设置count
        default:
            return state
    }
}

const UPDATE_USERNAME = 'UPDATE_USERNAME'
function userReducer(state = userInitialState, action){
    switch(action.type){
        case UPDATE_USERNAME:
            return {
                ...state,
                username: action.newName
            }
        default:
            return state
    }
}

const allReducers = combineReducers({
    counter: counterReducer,
    user: userReducer
})
const store = createStore(allReducers, {
    counter: initialState,
    user: userInitialState
})

//创建一个add函数，返回action
function add(num){
    return {
        type: ADD,
        num
    }
}

//store.dispatch({ type: ADD })
store.dispatch(add(3))
store.dispatch({ type: UPDATE_USERNAME, newName: 'lilei' })

console.log('合并的state', store.getState())
export default store
```
![](https://upload-images.jianshu.io/upload_images/5541401-573416b9297583ca.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
可以看到打印的数据 count 是4 了；
#####异步的action
使用 redux-thunk 插件来实现，通过 redux 的 applyMiddleWare 来使用中间件，将applyMiddleWare(ReduxThunk) 作为 store 第三个参数传入，就可以使用异步的 action 了；
安装 redux-thunk，它返回一个执行异步调度的函数，用于延迟动作的发送，内部函数接收 store 的 dispatch 
和 getState 作为参数
```
npm install redux-thunk --save-dev
```
实现一个异步的 action，asyncAdd 方法
```
import { createStore, combineReducers, applyMiddleware } from 'redux'
import ReduxThunk from 'redux-thunk'

const initialState = {
    count: 0
}

//user模块
const userInitialState = {
    username: 'john',
    age: 28,
    address: 'shanghai'
}

const ADD = 'ADD'
function counterReducer(state = initialState, action){
    // console.log(state, action)
    switch(action.type){
        case ADD:
            return { count: action.num + 1 }
        default:
            return state
    }
}

const UPDATE_USERNAME = 'UPDATE_USERNAME'
function userReducer(state = userInitialState, action){
    switch(action.type){
        case UPDATE_USERNAME:
            return {
                ...state,
                username: action.newName
            }
        default:
            return state
    }
}

const allReducers = combineReducers({
    counter: counterReducer,
    user: userReducer
})
const store = createStore(allReducers, {
    counter: initialState,
    user: userInitialState
}, applyMiddleware(ReduxThunk))

function add(num){
    return {
        type: ADD,
        num
    }
}

//实现一个异步的 action creators
function asyncAdd(){
    return (dispatch, getState) => {
        new Promise((resolve, reject) => {
            resolve(5)
        })
        .then((res) => {
            dispatch(add(res))
        })
        .then(() => {
            console.log('state changed:', getState())
        })
    }
}

//store.dispatch({ type: ADD })
store.dispatch(add(3))
store.dispatch(asyncAdd())
store.dispatch({ type: UPDATE_USERNAME, newName: 'lilei' })

export default store
```
![](https://upload-images.jianshu.io/upload_images/5541401-46213915c185376b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
thunk 做的就是将 dispatch, getState 传入，在 dispatch 之后，异步返回之后拿到最新的 state

####react-redux 连接 React 和 Redux
react-redux 是 React 官方指定的 Redux 插件，它允许 React 组件可以从 Redux 的 store 中读取数据，并且可以分发 action 到 store 去更新数据。
安装 react-redux
```
npm install react-redux --save-dev
```
react-redux 提供了 Provider 来使 redux 的 store 可以用到应用程序中，还提供了 connect 方法连接组件和 store
实例代码如下，将 Layout 组件用 Provider 包起来，传入 store
```
//components/Layout/Layout.js
import { hot } from 'react-hot-loader/root'
import React from 'react'

import { Provider } from 'react-redux'
import store from '../../components/Todolist/store.js'

import { Layout, Menu, Breadcrumb, Icon } from 'antd'
import Header from '../Header/Header'
import Footer from '../Footer/Footer'
import './layout.less'

const { Content, Sider } = Layout
const { SubMenu } = Menu

class LayoutContainer extends React.Component {
    state = {
        collapsed: false,
    }

    onCollapse = collapsed => {
        console.log(collapsed);
        this.setState({ collapsed });
    };

    render() {
        return (
            <Provider store={store}>
                <Layout style={{ minHeight: '100vh' }}>
                    <Header />           
                    <Layout>
                        ···
                    </Layout>
                </Layout>
            </Provider>
            
        );
    }
}

export default hot(LayoutContainer)
```
然后 connect Header 组件，connect 接收两个参数 `mapStateToProps`，`mapDispatchToProps`；
* `mapStateToProps`：每次 store 中的 state 改变时都会被调用，接收参数为整个的 state, 返回 React 组件所需的数据对象；
* `mapDispatchToProps`：此参数可以是函数，也可以是对象。如果参数是函数，参数为 dispatch，返回一个对象，对象里面的属性的类型是函数，此函数通过 dispatch 方法来 dispatch actions，它在组件创建的时候调用一次。
如果参数是对象，这个对象是一个 action creators，每一个 action creator 都会进入 props 函数，在调用的时候自动 dispatch action
```
//components/Header/Header.js
import React, { useState } from 'react'
import { connect } from 'react-redux'

import { Layout, Button } from 'antd'
const { Header } = Layout

import './header.less'

/*export default () => {
    //const [ username, setUsername ] = useState('john');
    return (
        <Header className="app-header">
            压制系统
        </Header>
    )
}*/

const HeaderBar = ({ counter, username }) => {
    return <Header>counter： {counter}，username：{username}</Header>
}
//mapStateToProps函数返回一个对象,将这个对象作为props传给Header组件
const mapStateToProps = (state) => {
    return {
        counter: state.counter.count,
        username: state.user.username
    }
}

const mapDispatchToProps = () => {
    return {
        
    }
}

export default connect(
    mapStateToProps,
    mapDispatchToProps
)(HeaderBar)
```
然后看到页面正确获取了 state;
![](https://upload-images.jianshu.io/upload_images/5541401-5f043705f714e4a7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
现在给 Header 组件添加一个按钮用来改变 state，然后更新 state 到视图
```
//components/Header/Header.js
import React, { useState } from 'react'
import { connect } from 'react-redux'

import { Layout, Button } from 'antd'
const { Header } = Layout

import './header.less'

/*export default () => {
    //const [ username, setUsername ] = useState('john');
    return (
        <Header className="app-header">
            压制系统
        </Header>
    )
}*/

const HeaderBar = ({ counter, username, rename, add }) => {
    return <Header>
        counter： {counter}，username：{username}
        <button onClick={() => add(counter + counter)}>add</button>
        <input value={username} onChange={(e) => rename(e.target.value)}/>
    </Header>
}

//mapStateToProps, 在每次store中的state改变时都会被调用，接收参数为整个的state, 返回Header组件所需的数据对象，这个对象会作为props传给Header组件
const mapStateToProps = (state) => {
    return {
        counter: state.counter.count,
        username: state.user.username
    }
}

const mapDispatchToProps = (dispatch) => {
    return {
        add: (num) => dispatch({ type: 'ADD', num}),
        rename: (newName) => dispatch({type: 'UPDATE_USERNAME', newName})
    }
}

//Connecting the Components
export default connect(
    mapStateToProps,
    mapDispatchToProps
)(HeaderBar)
```
可以看到 input 中输入内容，username 改变了，点击 add 按钮，counter 也改变了。

参考：
https://redux.js.org/introduction/getting-started
https://www.redux.org.cn/docs/introduction/Ecosystem.html
https://react-redux.js.org/introduction/basic-tutorial
https://www.npmjs.com/package/redux-thunk
https://react-redux.js.org/
