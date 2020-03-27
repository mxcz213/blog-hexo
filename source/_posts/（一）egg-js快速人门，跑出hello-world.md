---
title: （一）egg-js快速人门，跑出hello-world
---
一：什么是egg.js?

egg.js是nodejs 的一个框架，是基于koa框架的基础上整合的一套上层框架，既定的目录结构，开发者只需要基于MVC模式，根据项目规范的目录结构，专注于编写相应的controller,service,router,view,config配置，以及plugin插件可扩展。

二：脚手架快速生成项目

1.系统和环境：windows + node 8以上

2.脚手架命令，快速初始化生成项目

```

npm i egg-init -g
egg-init egg-myProject --type=simple
cd egg-myProject
npm i

```

3.启动项目

```

npm run dev

```

4.浏览器中打开http://127.0.0.1:7001：

![image](http://upload-images.jianshu.io/upload_images/5541401-c1513a29035577df?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

三：不用脚手架，逐步搭建，熟悉项目目录结构

1.初始化目录结构

```

mkdir egg-my-example
cd egg-my-example
npm init

```

npm init之后一路回车即可，然后安装egg  egg-bin:

```

npm i egg --save
npm i egg-bin --save-dev

```

2.找到package.json文件添加

```

"scripts": {

    "dev": "egg-bin dev"

  },

```

3.编写controller文件，router路由，添加配置文件config

在项目 egg-my-example 目录下新建app文件夹，新建controller文件夹，新建文件home.js

在项目 egg-my-example 目录下app文件夹新建router.js文件

在项目 egg-my-example 目录下新建与app同级的config文件夹，（**注意：config文件夹跟app同级目录**），新建config.default.js文件

```

//app/controllter/home.js

const Controller = require('egg').Controller;

class HomeController extends Controller{

    async index(){

        this.ctx.body = "hello world";

    }

}

module.exports = HomeController;

```

```

//app/router.js

module.exports = app => {

  const { router, controller } = app;

  router.get('/',controller.home.index);

  router.get('/list',controller.news.list);

}

```

```

//config/congif.default.js

exports.keys = '123456790';  //key是自己的cookie信息

```

整体项目目路结构如下：

![image](http://upload-images.jianshu.io/upload_images/5541401-0ecc46ba30717a76?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

4.运行项目

```

npm run dev

```

5.浏览器打开   http://127.0.0.1:7001

![image](http://upload-images.jianshu.io/upload_images/5541401-9b6d73ffd42914a5?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

如此一个简单的hello world就完成了，手动新建项目过程中一定要注意目路结构，egg根据既定的项目目录，开发者可以专注编写controller等业务代码，快速完成项目。

6.接下来可以继续编写view模板文件
使用 Nunjucks来渲染，安装对应的插件 egg-view-nunjucks
```
npm i egg-view-nunjucks --save
```
开启插件：（config目录下新建plugin.js文件）
```
//config/plugin.js
exports.nunjucks = {
    enable:true,
    package:'egg-view-nunjucks'
}
```
添加view模板配置
```
//config/config.default.js
exports.view = {
    defaultViewEngine:'nunjucks',
    mapping:{
        '.tpl':'nunjucks'
    }
}
```
7.为一个列表页编写模板文件
在app目录下新建一个view文件夹，将所有模板文件放到view下
```
//app/view/news/list.tpl
<html>
  <head>
    <title>Hacker News</title>
    <link rel="stylesheet" href="/public/css/news.css" />
  </head>
  <body>
    <ul class="news-view view">
      {% for item in list %}
        <li class="item">
          <a href="{{ item.url }}">{{ item.title }}</a>
        </li>
      {% endfor %}
    </ul>
  </body>
</html>
```
添加controller,router
```
//app/controller/news.js
const Controller = require('egg').Controller;
class NewsController extends Controller {
   async list() {
    const dataList = {
      list: [
        { id: 1, title: 'this is news 1', url: '/news/1' },
        { id: 2, title: 'this is news 2', url: '/news/2' }
      ]
    };
    await this.ctx.render('news/list.tpl', dataList);
}
module.exports = NewsController;
```
```
//app/router.js
module.exports = app => {
   const { router, controller } = app; 
   router.get('/',controller.home.index);
   router.get('/list',controller.news.list);
}
```
8.浏览器中打开http://127.0.0.1:7001/list
![image.png](https://upload-images.jianshu.io/upload_images/5541401-f58625a0f97b7553.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


参考：[https://eggjs.org/zh-cn/intro/quickstart.html](https://eggjs.org/zh-cn/intro/quickstart.html)
