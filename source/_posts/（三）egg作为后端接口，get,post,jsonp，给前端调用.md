---
title: （三）egg作为后端接口，get,post,jsonp，给前端调用
---
上一篇讲了前后端分离，`egg`作为接口段提供接口服务，现在说一下最基本的，常用的几个请求，`post，get，jsonp`请求该如何做。

**（一）get请求**

打开上一篇文章讲到的`egg`项目`（ egg-my-example）`，找到`router`文件，添加如下路由：

```
//app/router.js
module.exports = app => {
   const { router, controller } = app; 
   router.get('/',controller.home.index);
   router.get('/list',controller.news.list);
}
```
此时新建一个`/list`接口，到`controller`里面找到`news`文件，新建一个`list`的方法
```
//app/controller/news.js
const Controller = require('egg').Controller;
class NewsController extends Controller {
    async list() {
        const dataList = await this.other();
        this.ctx.body = {
            code:0,
            masg:'news list success',
            data:dataList
        };
    }

    async other() {
        return {
            list: [
                { id: 1, title: 'this is news 1', url: '/news/1' },
                { id: 2, title: 'this is news 2', url: '/news/2' }
            ]
        }
    }
}
module.exports = NewsController;
```
现在启动项目：`npm run dev`，看到如下请求数据返回：
![](https://upload-images.jianshu.io/upload_images/5541401-40352faa76134e6b.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
此时接口已经准备就绪，切到上一个前端项目里面`http-server-test`，在这里请求`/list`接口：

```
//因为用到了async,await，所以配置babel的presets的stage-0
//.babelrc
{
    "presets":["es2015","react","stage-0"]
}
```
```
//components/hello.jsx
import React from 'react'
import './hello.less'
import axios from 'axios'

export default class HelloComponent extends React.Component{
	constructor(props){
		super(props);
		this.state = {
			listData:[]
		}
	}

	async componentDidMount(){
		await axios.get('/list')
		.then((res) => {
			const listData = res.data.data.list;
			console.log(listData);
			this.setState({
				listData
			});
		});
	}

    render(){
    	const listData = this.state.listData;
    	let ele = listData.map((item) => {
    		return <p key={item.id}>{item.title}:{item.url}</p>
    	})
        return(
            <div className="hello-box">
            	<div className="hello-list">{ele}</div>
            </div>
        );
    }
}
```
如此一个`get`请求结束了，现在打包我们的前端项目，运行`npm run build`将打包好的项目`js,css`链接复制到`egg`项目的/路径下的`view`里面，即如下代码：
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
最后打开浏览器输入`http://127.0.0.1:7001/`，即可看到效果：
![](https://upload-images.jianshu.io/upload_images/5541401-b7f91ca0d14b573d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一个`get`请求就做完了。

（二）如法炮制post请求
```
//app/router.js
module.exports = app => {
   const { router, controller } = app; 
   router.get('/',controller.home.index);
   router.get('/list',controller.news.list);
   router.post('/form',controller.form.post);
}
```
```
//app/controller/form.js
const Controller = require('egg').Controller;
class FormController extends Controller {
    async post(){
        this.ctx.body = {
            code:0,
            masg:'form submit success'
        }
    }
}
module.exports = FormController;
```
切到前端项目http-erver-test中
```
//components/hello.jsx
import React from 'react'
import './hello.less'
import axios from 'axios'

export default class HelloComponent extends React.Component{
	constructor(props){
		super(props);
		this.state = {
			listData:[]
		}
	}

	async componentDidMount(){
		await axios.get('/list')
		.then((res) => {
			const listData = res.data.data.list;
			console.log(listData);
			this.setState({
				listData
			});
		});
	}

	submitForm(){
		axios.post('/form',{
			hello:document.getElementById('input').value,
			now:new Date().getTime()
		})
		.then((res) => {
			alert(res.data.masg);
		})
		.catch((err) => {
			console.log(err);
		})
	}

    render(){
    	const listData = this.state.listData;
    	let ele = listData.map((item) => {
    		return <p key={item.id}>{item.title}:{item.url}</p>
    	})
        return(
            <div className="hello-box">
            	<div className="hello-list">{ele}</div>
            	<form>
            		<input type="text" name="hello" id="input" />
            		<input type="button" onClick={this.submitForm} value="提交" />
            	</form>
            </div>
        );
    }
}
```
最后再重新npm run build打包http-server-test项目即可:
![](https://upload-images.jianshu.io/upload_images/5541401-0e70a6bf69394591.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
一个post请求就完成了。

（三）jsonp请求
```
//app/router.js
module.exports = app => {
   const { router, controller } = app; 
   router.get('/',controller.home.index);
   router.get('/list',controller.news.list);
   router.post('/form',controller.form.post);
   router.get('/demo/count.json',app.jsonp(),controller.demo.count);
}
```
```
// app/demo/count.json
{
    "errcode":"0",
    "msg":"success",
    "data":[
        {
            "id":1,
            "title":"hello"
        },
        {
            "id":2,
            "title":"world"
        }
    ]
}
```
```
// app/controller/demo.js
const Controller = require('egg').Controller;
class DemoJsonp extends Controller{
    async count(){
        this.ctx.body = {
            data:[
                {
                    "id":1,
                    "title":"hello"
                },
                {
                    "id":2,
                    "title":"world"
                }
            ]
        };
    }
}
module.exports = DemoJsonp;
```
其实我们这里并没有跨域，只是类比一下，跟get其实一样。

![](https://upload-images.jianshu.io/upload_images/5541401-a39079ffe455ef4a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这一阶段就结束了，由浅到深，慢慢来：）











