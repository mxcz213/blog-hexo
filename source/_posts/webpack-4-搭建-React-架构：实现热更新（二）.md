---
title: webpack-4-搭建-React-架构：实现热更新（二）
---
上一篇文章 [webpack 4 搭建React antd 中后台项目架构](https://www.jianshu.com/p/846001b37a83) 实现了基本的架构工作，让项目可以跑起来，虽说依照 webpack 中文官网，在 webpack.dev.js 中配置了 hot，只有修改样式的时候才能热更新，可以像在浏览器中修改样式一样的快速，修改 js 文件依然没有效果，主要是原因是 style-loader 实现了HMR接口，而 react 的 js 文件并没有实现这个功能：  

```
//webpack.dev.js
...
devServer: {
        contentBase: path.resolve(__dirname, 'dist'),
+        hot: true,
        hotOnly: true,
        open: false,  //自动打开浏览器
        port: 9000,
        overlay: {
            warnings: false,
            errors: true
        }
    },
    plugins: [
+        new webpack.NamedModulesPlugin(),
+        new webpack.HotModuleReplacementPlugin()
    ],
...
```
```
//main.js
import React from 'react'
import ReactDOM from 'react-dom'
import App from './app'
import Layout from './components/Layout/Layout'
import './css/main'
import './css/base'

ReactDOM.render(
    //<App />,
    <Layout />,
    document.getElementById('app')
)

+ if(module.hot){
+     module.hot.accept();
+ }
```
猜想可能是引入 react 的缘故，于是重新加一个入口文件 `index.js` 来使验证一下
```
//webpack.common.js
...
entry: {
  app: './src/index.js'
}
...
```
```
//入口文件 index.js
import { square } from './math.js'
console.log('打印2的平方',square(2))

let arr = [1,2,3];
let arr2 = arr.map(item => item + 2)
arr.includes(8)
console.log('new Set(arr2):', new Set(arr2))

async function l() {
    return await 1;
}
l().then((value) => {
    console.log(value);
    console.log(111111)
})

const app = document.getElementById('app')
app.innerHTML = 'hello world';

+ if(module.hot){
+     module.hot.accept();
+ }
```
这样在启动项目，刷新页面，可以看到直接修改 `index.js` 中的 html 内容，页面实时更新了。由此可见 react 框架并没有做热更新，以前用 vue-cli 创建项目的时候，是可以进行热更新的，原因是 vue-loader 实现了这个功能，而 react 没有做到这样，所有我们需要借助插件来实现，插件就是 `react-hot-loader`。

现在通过 `react-hot-loader` 来实现 react 的热更新：
* 安装最新版的 react-hot-loader；
```
cnpm install @hot-loader/react-dom --save-dev
```
* 将入口文件换成 main.js
```
//webpack.common.js
...
entry: {
  app: './src/main.js'
}
...
```
* 修改 root component （src/components/Layout/Layout.js），在 react react-dom 之前引入 `react-hot-loader/root`，然后将组件用 `hot` 包起来；
```
+ import { hot } from 'react-hot-loader/root'
import React from 'react'
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
        <Layout style={{ minHeight: '100vh' }}>
            <Header />           
            <Layout>
                <Sider collapsible collapsed={this.state.collapsed} onCollapse={this.onCollapse}>
                    <Menu theme="dark" defaultSelectedKeys={['1']} mode="inline">
                        <Menu.Item key="1">
                        <Icon type="pie-chart" />
                        <span>Option 1</span>
                        </Menu.Item>
                        <Menu.Item key="2">
                        <Icon type="desktop" />
                        <span>Option 2</span>
                        </Menu.Item>
                        <SubMenu
                        key="sub1"
                        title={
                            <span>
                            <Icon type="user" />
                            <span>User</span>
                            </span>
                        }
                        >
                        <Menu.Item key="3">Tom</Menu.Item>
                        <Menu.Item key="4">Bill</Menu.Item>
                        <Menu.Item key="5">Alex</Menu.Item>
                        </SubMenu>
                        <SubMenu
                        key="sub2"
                        title={
                            <span>
                            <Icon type="team" />
                            <span>Team</span>
                            </span>
                        }
                        >
                        <Menu.Item key="6">Team 1</Menu.Item>
                        <Menu.Item key="8">Team 2</Menu.Item>
                        </SubMenu>
                        <Menu.Item key="9">
                        <Icon type="file" />
                        <span>File</span>
                        </Menu.Item>
                    </Menu>
                </Sider>
                <Content style={{ margin: '0 16px' }}>
                    <Breadcrumb style={{ margin: '16px 0' }}>
                        <Breadcrumb.Item>User</Breadcrumb.Item>
                        <Breadcrumb.Item>Bill</Breadcrumb.Item>
                    </Breadcrumb>
                    <div style={{ padding: 24, background: '#fff', minHeight: 360 }}>Bill is a cat.</div>
                    <Footer />
                </Content>
            </Layout>
        </Layout>
        );
    }
}

+ export default hot(LayoutContainer)
```
* 继续运行项目，如图，已经可以实时更新了，并将修改的文件路径也给打印出来了；
![](https://upload-images.jianshu.io/upload_images/5541401-a50f47d451f0c069.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

可以看到控制台出现了警告，说是 `react-dom pacth` 没有检测到，可能会影响我们使用 `react 16.6` 以上版本的特性，比如 `hook`，查看 `react-hot-loader` 配置，安装 `@hot-loader/react-dom` 这个插件，将其指到 react-dom 即可；在 webpack.common.js 中作如下配置：
```
//webpack.common.js
...
resolve: {
        extensions: ['.js', '.css', '.less'],
        alias: {
            'react-dom': '@hot-loader/react-dom'
        }
    },
...
```
最后运行项目，这个警告就没有了；
![](https://upload-images.jianshu.io/upload_images/5541401-32c6b9118806b502.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最后我们来验证一下用 `hook` 特性来写 `Header.js`，看一下 `hot` 是不是起作用；修改 `Header.js` 为一个函数组件，然后使用 hook，来使用 react class 的 state 特性；
```
//src/components/Header/Header.js
import React, { useState } from 'react'
import { Layout, Button } from 'antd'
const { Header } = Layout

import './header.less'

export default () => {
    const [ username, setUsername ] = useState('john');
    return (
        <Header className="app-header">
            <span>我的名字叫 {username}</span>
            <Button type="primary" onClick={() => setUsername('lyli')}>修改名字</Button>
        </Header>
    )
}
```
我们在 Header.js 中引入 `useState` hook，新建一个变量和设置变量的函数，初始值是 `john`，点击按钮修改为 `lyli`，可以看到页面直接更新了，并没有重新刷新；
![](https://upload-images.jianshu.io/upload_images/5541401-8f2e28a34109899b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

最终我们实现了 react 项目的热更新，项目地址在这里，大家可以下载尝试一下：https://github.com/mxcz213/webpack4-demo

参考：
https://webpack.js.org/guides/hot-module-replacement/
https://www.npmjs.com/package/react-hot-loader
