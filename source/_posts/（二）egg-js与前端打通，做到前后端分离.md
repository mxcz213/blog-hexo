---
title: （二）egg-js与前端打通，做到前后端分离
---
**此文的目的是：让egg作为中间接口层，只出接口，前端做页面渲染，搞成前后端分离。**

思路是：首先给前端启动一个服务，用`http-server` 或者 `webpack-dev-server`
              后端启动一个服务，将前端代码打包之后的js,css文件通过script，link的方式引入到egg的模板文件里去。
**一：http-server  启动一个前端服务**

新建一个前端项目：
```
mkdir http-server-test
cd http-server-test
npm install http-server -g
```
新建一个`test.html`页面，随便写个hello world,然后运行
```
http-server
```
访问`http://127.0.0.1:8080/test.html` 得到下面页面
![](https://upload-images.jianshu.io/upload_images/5541401-1359667d1e1b6583.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
前端`http server`就启动好了。

**二：前端项目通过webpack工具+react前端框架打包出js,css文件**

1.进到http-server-test目录,`npm init`一路回车即可
```
npm init
```
2.安装相关依赖
```
//babel编译js相关
npm install babel-loader babel-core babel-preset-es2015 babel-preset-react --save-dev
```
```
//css文件处理相关
npm install css-loader style-loader less-loader postcss-loader less --save-dev
```
```
//将css达成独立的css文件
//加上@next是为了防止出错，遇到了这个坑
npm install extract-text-webpack-plugin@next --save-dev
```
```
//安装react框架相关
npm install react react-dom --save 
```
```
//webpack打包文件相关依赖
npm install webpack -g
npm install webpack --save-dev
//因为直接webpack打包的时候报错说是webpack-cli是独立出来的，所以也要安装一下
npm install webpack-cli --save-dev       
```
3.配置babel,新建一个`.babelrc`文件，写入如下代码
```
{
    "presets":["es2015","react"]
}
```
4.配置webpack文件，新建`webpack.config.js`文件，写入如下代码：
```
const webpack = require('webpack');
const path = require('path');
const ExtractTextPlugin = require('extract-text-webpack-plugin');
module.exports = {
    context:path.resolve(__dirname,''),
    entry:{
        main:'./src/index.jsx'
    },
    output:{
        path:path.resolve(__dirname,'dist/'),
        filename:'index.js'
    },
    module:{
        rules:[
            {
                test:/\.jsx$/,
                use:'babel-loader'
            },
            {
                test:/\.less$/,
                use: ExtractTextPlugin.extract({
                    fallback: "style-loader",
                    use: ["css-loader","less-loader"]
                  })
            }
        ]
    },
    plugins:[
        new webpack.DefinePlugin({
            'process.env.NODE_ENV':JSON.stringify('development'),
            __DEV__:true
        }),
        new ExtractTextPlugin('index.css'),
    ]
}
```
5.继续丰富目录，目路结构（`dist`目录是`webpack`打包自动生成的文件，不需要手动创建）如下：
![](https://upload-images.jianshu.io/upload_images/5541401-1db7ffea8265f5b4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
//src/index.html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>hello</title>
</head>
<body>
    <div id="app"></div>
    <script src="../dist/index.js"></script>
</body>
</html>
```
```
//src/index.jsx
import React from 'react'
import ReactDOM from 'react-dom'
import HelloComponent from '../components/hello.jsx'
import './index.less'

ReactDOM.render(
    <div className="app-box">
        <HelloComponent />
    </div>,
    document.getElementById('app')
);
```
```
//src/index.less
.app-box{
    width:200px;
    height: 400px;
    margin:50px auto;
    border:1px solid #ccc;
}
```
```
//components/hello.jsx
import React from 'react'
import './hello.less'

export default class HelloComponent extends React.Component{
    render(){
        return(
            <div className="hello-box">hello world</div>
        );
    }
}
```
```
//components/hello.less
.hello-box{
    color:#f00;
}
```
6.运行webpack打包出文件
```
webpack --watch
```
![](https://upload-images.jianshu.io/upload_images/5541401-97ad22738e484256.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**三：将打包出来的css，js文件通过script src=""链接到egg.js的模板文件中**

打开上一节新建的egg项目
![](https://upload-images.jianshu.io/upload_images/5541401-8fc3f058914e0b1e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
//app/controller/home.js
const Controller = require('egg').Controller;
class HomeController extends Controller{
    async index(){
       await this.ctx.render('home/home.tpl');
    }
}
module.exports = HomeController;
```
```
//app/view/home/home.tpl
<html>
  <head>
    <title>Hacker News</title>
    <link rel="stylesheet" href="http://127.0.0.1:8080/dist/index.css" />
  </head>
  <body>
    <div id="app"></div>
    <script src="http://127.0.0.1:8080/dist/index.js"></script>
  </body>
</html>
```
最后命令行运行：
```
npm run dev
```
访问`http://127.0.0.1:7001/`
![](https://upload-images.jianshu.io/upload_images/5541401-10c475e52b8b1090.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5541401-a7d5744f6ab71544.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

**注意：前端的服务http-server,以及egg的服务都要开启。**

结语：如此便实现egg.js负责接口服务，前端做前端的事，前后端分离。
