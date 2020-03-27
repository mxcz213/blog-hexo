---
title: 线上分享webpack（每人一元）总结：资源加载
---
#####1.clone webpack代码到本地
```
const path = require('path');
module.exports = {
    devtool: 'source-map',      //配置source map，查看源文件
    entry: {                              //构建入口
        index: path.resolve(__dirname, 'src/index.js'),
    },
    output: {                           //构建输出到dist目录
        devtoolModuleFilenameTemplate: '[resource-path]',     //配置source map在浏览器中的展现方式
        path: path.resolve(__dirname, 'dist/'),
    },
    module: {      //配置loader，解析相应的文件
        rules: [
            { test: /\.js$/, use: { loader: 'babel-loader', query: { presets: ['@babel/preset-env'] } } },
        ]
    },
};
```
![](https://upload-images.jianshu.io/upload_images/5541401-a503b4db8abe3b19.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
生成的`source map`文件
![](https://upload-images.jianshu.io/upload_images/5541401-1976ea2c3124540e.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####2.启动调试项目
```
npm install    
```
如果安装过程中出错，使用`管理员`身份再重新使用安装
![image.png](https://upload-images.jianshu.io/upload_images/5541401-5d70f55a86c2f268.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`package.json`里有用来调试的`script`，直接在`vscode`中按`F5`就可以启动调试了，
`windows`用户会报错：
![](https://upload-images.jianshu.io/upload_images/5541401-9dadd65f6822ee3d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
，`windows`用户需要把这两条行改成
```
 "debug:build": "node --inspect-brk=5858 node_modules/webpack/bin/webpack.js",
 "debug:watch": "node --inspect-brk=5859 node_modules/webpack --watch"
```
因为`webpack`是直接安装在`node_modules`下的。
#####3.调试`webpack`资源加载流程
修改之后再按`F5`启动调试，进入到`node_modules/webpack/bin/webpack.js`
![](https://upload-images.jianshu.io/upload_images/5541401-048340c05a0d0d0a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这个文件的逻辑是判断一下`cli`是否安装了，然后去调`webpack.run`；
然后在`150行`打上断点，按`F5`，这里去调`webpack-cli`去了，然后`webpack-cli`又调回到`webpack/lib`，
![](https://upload-images.jianshu.io/upload_images/5541401-1e01bb1787538324.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
后面我们不去看`weboack-cli`，直接在`webpack/lib/Compiler.js`的`199行`打个断点
![](https://upload-images.jianshu.io/upload_images/5541401-0fa9af34d6fbe129.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
#####总体来说`webpack build`做了这几件事：
######1.载入资源，这个过程会调用各个`loader`，然后用`loader`去载入资源文件，涉及到的模块是`loader-runner`
######2.代码压缩，这里只看`js`文件，涉及到的模块是`uglifyjs-webpack-plugin`
######3.代码生成，就是在`Compiler.js`中，`emit`了一下，存到文件中

接下来看下`compile`函数，跟一下资源加载过程，在`536行`打个断点
![](https://upload-images.jianshu.io/upload_images/5541401-00dbef693fd5b1e6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里会调用`make这个hook`，`webpack`就是用`hook`这种东西实现了`插件化`，也就是`webpack`只负责调一下，至于哪个插件实现了`make这个hook`，`webpack`不管的，`单步调试`进去看一下
![](https://upload-images.jianshu.io/upload_images/5541401-d7bc59f0a51045b1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
跑到了`tapable`这个库中，这个`tapable`就是`webpack`实现`hook`的模块，`tapable`读起来比较费力，它是用生成代码的方式做的，这个应该是兼容性和功能性的考虑，因为`es6`的`proxy`能力有限，虽然能做一些拦截，但是做不到`hook`这么强大也不好搞。
继续，按`单步跳过（F10）`到下一行，
![](https://upload-images.jianshu.io/upload_images/5541401-87526c89c4571fc2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
然后`单步调试（F11）`,就会跑到这里来，这个`VM1372`，就是`tapable`临时生成的代码，一般出现这种代码的时候，就意味着有人用了`new Function`这种东西，`动态生成函数`
![](https://upload-images.jianshu.io/upload_images/5541401-1f28b5dc956a8d13.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
接下来`单步跳过`，`单步调试进入_fn0中`
![](https://upload-images.jianshu.io/upload_images/5541401-40401014e511e1f6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进去就到了`webpack/lib/SingleEntryPlugin.js`中的`第43行`，`compiler.hooks.make.tapAsync`(这个`SingleEntryPlugin.js`实现了`make这个hook`，然后`Compiler.js中this.make.hooks`调用时，就调到了。
![](https://upload-images.jianshu.io/upload_images/5541401-2d350417d3746875.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
######总结：这里主要是从`Compiler.js的compile`函数，看`webpack`怎样从`this.hooks.make.callAsync`调到`SingleEntryPlugin.js`的`compiler.hooks.make.tapAsync`实现的。
中间涉及到了`tapable`，他提供了一个功能，让`hook`可以在其他地方实现，`webpack`只需要调一下这个`hook`就行了。同样的`hook`，可以实现多次，`webpack`会放到一个队列中调用。`hook`还有很多种类，原理是一样的，只不过处理方式不同，比如异步的，同步的。

继续下去，现在到这里了，这里直接调用了`compilation.addEntry`，这是`webpack`的另一个重要的文件，`Compilation.js`
![](https://upload-images.jianshu.io/upload_images/5541401-bbad54913ffbdc11.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进入看看，在`1028行`打上断点，这里主要看这个文件，开始加载资源，
![](https://upload-images.jianshu.io/upload_images/5541401-b209e1ceb05e08ad.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`this._addModuleChain`的逻辑是这样的，先加载入口文件，然后再递归加载它依赖的文件，依赖的依赖的文件，就是通过`require的import`的那些。
![](https://upload-images.jianshu.io/upload_images/5541401-7224e346c663ec05.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
进去看`_addModuleChain`，开始加载入口文件了，往下看，在`943和953行`打上断点
![](https://upload-images.jianshu.io/upload_images/5541401-07fa417f15b2d147.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`module`出来了，包含了如下这些内容
![](https://upload-images.jianshu.io/upload_images/5541401-997e5bd85828fd32.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5541401-04adfdbce9885f8c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`request`是`loader`，`resource`是`待加载的资源`，`webpack`造出`module`后再调用这个`module`的`build`方法来加载资源，十分的`OO`，不直接干，而是先搞一个`factory`，让`factory`造`module`，再让`module`自己`build`
![](https://upload-images.jianshu.io/upload_images/5541401-40ab1ea0f4f32607.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这里开始`build`
![](https://upload-images.jianshu.io/upload_images/5541401-25416b5ac143d2e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5541401-82f8a7f5759df618.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
![](https://upload-images.jianshu.io/upload_images/5541401-d99ffd1fd492c918.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`this.buildModule` -> `module.build` -> `doBuild`
最后到了`NormalModule.js`的`265行`调了`runLoaders`，从`loader-runner`里`require`进来的
```
const { getContext, runLoaders } = require("loader-runner");
```
进去看看
![](https://upload-images.jianshu.io/upload_images/5541401-3f9170a3eb71bfd6.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

`loader-runner`首先会载入那个`loader`，然后在用`loader`加载资源，例子中的是`babel-loader`，资源文件就是`src/index.js`，这个`LOADER_EXECUTION`就是用`loader`载入资源用的
![](https://upload-images.jianshu.io/upload_images/5541401-cf9b10ffaad1d8e2.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
`fn`是`loader,args`是源文件的内容，，然后`webpack`调用这个`loader`，`babel-loader`可以看做一个函数把`ES6`代码转译成`ES5`。
以上就是解析入口文件，一图胜千言
![](https://upload-images.jianshu.io/upload_images/5541401-f0e1e57a1dc56c18.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

###参考
webpack debug：https://github.com/thzt/debug-webpack
webpack群侠传系列：https://www.jianshu.com/p/de262ad255c3
深入浅出webpack：https://wangchong.tech/webpack/
webpack4.0官方文档：https://webpack.js.org/concepts
