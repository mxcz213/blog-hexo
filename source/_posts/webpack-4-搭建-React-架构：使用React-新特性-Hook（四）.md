---
title: webpack-4-搭建-React-架构：使用React-新特性-Hook（四）
---
####简介
Hook 是`React 16.8` 的新增特性，它可以让你在不编写 class 的情况下，使用 state 以及其他 React 特性。

以前我们在写 React 组件的时候，如果是纯展示型组件，从上层接收数据显示，这时候我们是通过函数组件的方式来写，通过 props 将数据展示出来。函数组件本身是没有生命周期和实例方法的，所以不能管理自身的 state，如果想要使用 state，只能通过类组件来改写。Hook 就是来解决这个问题，让函数组件可以操作自身的 state 以及应用其他的 React 特性。

####什么是Hook
Hook 是一些可以让我们在函数组件里`钩入` React state 及生命周期等特性的函数。这使得我们不使用 class 组件也能使用 React。所以我们可以在新组件中开始使用 Hook。

####Hook 概览

* **State Hook：useState**
一个简单的演示：
```
import React, { useState } from 'react'
function Example() {
  const [count, setCount] = useState(0)
  return(
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
   )
}
```
useState 参数是初始的 state，返回的是一对值，当前状态，以及更新状态的函数，如上代码。
还可以在一个组件中多次使用 State Hook
```
function ExampleWithManyStates() {
  // 声明多个 state 变量！
  const [age, setAge] = useState(42);
  const [fruit, setFruit] = useState('banana');
  const [todos, setTodos] = useState([{ text: 'Learn Hooks' }]);
  // ...
}
```
* **Effect Hook：useEffect**
之前我们在 React 组件中执行过数据的获取，订阅，手动修改过 DOM等，这些我们统一称之为副作用，或者说是作用。

useEffect 就是一个 Effect Hook，给函数组件增加操作副作用的能力。它跟 class 组件中的componentDidMount、componentDidUpdate 和 componentWillUnmount 具有相同的用途，只不过被合并成了一个 API。

现在我们使用 useEffect 来写一个组价在 React 更新 DOM 后设置一个页面标题
```
import React, { useState, useEffect } from 'react';

function Example() {
  const [count, setCount] = useState(0);

  // 相当于 componentDidMount 和 componentDidUpdate:
  useEffect(() => {
    // 使用浏览器的 API 更新页面标题
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
useEffect 会在第一次渲染之后和每次状态更新后都执行。在某些情况下我们不需要每次更新之后都执行 effect，可以设置 useEffect 的第二个参数：
```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, [count]); // 仅在 count 更改时更新
```
如果想执行只运行一次的 effect，可以传递一个空数组：
```
useEffect(() => {
  document.title = `You clicked ${count} times`;
}, []); //只在组件第一次挂载之后执行
```
####Hook 规则
Hook 本质就是 JavaScript 函数，但是使用的时候需要遵循两个规则：
* 只在最顶层使用 Hook（不要在循环，条件或者嵌套函数中调用 Hook）
* 只在 React 函数组件中调用 Hook（不要在普通的 JavaScript 函数中调用 Hook）

--------------------------------------------------分割线------------------------------------------------------

前面介绍完了 Hook，现在在我们的项目中用起来，尝试一下吧：）
安装 react-router，`npm install react-router react-router-dom --save-dev`，
接下来改造入口文件 main.js，不用 class，采用函数组件，使用 useState Hook
```
// src/main.js
import React, { useState } from 'react'
import ReactDOM from 'react-dom'
import { HashRouter, Route } from 'react-router-dom'
import Layout from './components/Layout/Layout'
import Home from './pages/Home'
import VodManage from './pages/VodManage'

const App = () => {
    const [selectedKeys, setSelectedKeys] = useState(['1'])

    return(
        <HashRouter basename="/">
            <Layout selectedKeys={selectedKeys} onChangeSelectKey={(keys) => setSelectedKeys(keys)}>
                <Route path="/" render={() => <Home onChangeSelectKey={(keys) => setSelectedKeys(keys)} />} exact />
                <Route path="/vodmanage" render={() => <VodManage onChangeSelectKey={(keys) => setSelectedKeys(keys)} />} />
            </Layout>
        </HashRouter>
    )
}

ReactDOM.render(
    <App />,
    document.getElementById('app')
)

if(module.hot){
    module.hot.accept();
}
```
用 useState 来保存切换路由的 keys `const [selectedKeys, setSelectedKeys] = useState(['home'])`
接下来在pages目录下写两个页面用来路由跳转，Home.js 和 VodManage.js，用useEffect来调用设置选中状态的路由keys，因为只有在第一次组件挂载之后执行一次即可，所以useEffect的第二参数设为空数组`useEffect(() => {},[]) `
```
// src/pages/Home.js
import React, { useEffect } from 'react'

const Home = ({ onChangeSelectKey }) => {
    const keys = ['home']

    useEffect(() => {
        onChangeSelectKey(keys)
    }, [])

    return (
        <div>
            home  home   home 
        </div>
    )
}
export default Home
```
VodManage.js 同样的写法
```
// src/pages/VodMange.js
import React, { useEffect } from 'react'

const VodManage = ({ onChangeSelectKey }) => {
    const keys = ['vodmanage']

    useEffect(() => {
        onChangeSelectKey(keys)
    }, [])

    return (
        <div>
            vod manage
        </div>
    )
}
export default VodManage
```
我们把左边的菜单栏单独抽出来一个组件，将左边栏数据保存在一个 canstant.js 文件中
```
// src/components/constant.js
export const leftMenu = [
    {
        title: 'Home',
        key: 'home',
        path: '/',
        iconType: 'pie-chart',
        subMenu: []
    },
    {
        title: '点播管理',
        key: 'vodmanage',
        path: '/vodmanage',
        iconType: 'desktop',
        subMenu: []
    }
]
```
左边栏组件 LeftMenu.js
```
// src/components/LeftMenu/LeftMenu.js
import React from 'react'
import { Link } from 'react-router-dom'
import { Menu, Icon } from 'antd'
import { leftMenu } from '../constant'

const MenuItem = Menu.Item
const SubMenu = Menu.SubMenu

const LeftMenu = ({ selectedKeys, onChangeSelectKey }) => {
    const constMenuStr = leftMenu.map((item, index) => {
        return item.subMenu.length === 0 
            ? 
                <MenuItem key={item.key}>
                    <Link to={item.path}>
                        <Icon type={item.iconType} />
                        <span>{item.title}</span>
                    </Link>
                </MenuItem>
            :   <SubMenu 
                    key={item.key} 
                    title={
                        <span>
                            <Icon type={item.iconType} />
                            <span>{item.title}</span>                       
                        </span>
                    }
                >
                    {
                        item.subMenu.map((sub) => {
                            return (
                                <MenuItem key={sub.key}>
                                    <Link to={sub.path}>
                                        <Icon type={sub.iconType} />
                                        <span>{sub.title}</span>
                                    </Link>
                                </MenuItem>
                            )
                        })
                    }
                </SubMenu>

    })
    return(
        <Menu 
            theme="dark"
            selectedKeys={selectedKeys}
            mode="inline" 
            onClick={({ item, key, keyPath, domEvent }) => onChangeSelectKey([key])}
        >
            { constMenuStr }
        </Menu>
    )
}

export default LeftMenu
```
可以看到页面可以很好的展示：
![](https://upload-images.jianshu.io/upload_images/5541401-865886e5ab2925c6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
更多的 Hook 的使用方法可以参考官网解读。这个例子只做入门。

参考：
https://react.docschina.org/docs/hooks-intro.html
