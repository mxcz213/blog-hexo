---
title: webpack-4-中-tree-shaking-生产环境配置
---
上一篇介绍了什么是`tree shaking`，这一篇我们来实操一下。
node 版本是` v8.9.1`，webpack的版本是`4.35.0`。

最终的项目文件目录结构如下：
![](https://upload-images.jianshu.io/upload_images/5541401-94d7de55e8f4bb2d.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

新建一个项目`webpack-4-demo`，然后`npm init -y`，生成`package.json`文件，然后安装`webpack 4`
```
 npm install --save-dev webpack webpack-cli webpack-dev-server webpack-merge
```
```
npm install --save-dev clean-webpack-plugin html-webpack-plugin
```
新建`src `目录作为源文件目录，新建webpack配置文件，区分开发环境和生产环境，然后使用`webpack-merge`来合并配置。
新建基础配置文件`webpack.common.js`，代码如下
```
//webpack.common.js
const path = require('path')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const HtmlWebpackPlugin = require('html-webpack-plugin')

module.exports = {
    entry: {
        app: './src/index.js',
    },
    plugins: [
        new CleanWebpackPlugin(),
        new HtmlWebpackPlugin({
            title: 'webpack 4 production',
            template: 'index.html',
            filename: 'index.html'
        }),
    ],
    output: {
        filename: '[name].bandle.js',
        path: path.resolve(__dirname, 'dist')
    },
    module: {
        rules: [
        ]
    }
}
```
新建开发环境配置文件`webpack.dev.js`
```
//webpack.dev.js
const path = require('path')
const merge = require('webpack-merge')
const common = require('./webpack.common.js')
const webpack = require('webpack')

module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        contentBase: './dist',
        hot: true,
        port: 9000
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(),   //启用HMR,配合server的hot
    ]
})
```
新建生产环境配置文件`webpack.prod.js `
```
const merge = require('webpack-merge')
const common = require('./webpack.common.js')

module.exports = merge(common, {
    devtool: 'source-map'
})
```
新建项目入口文件src/index.js
```
//src/index.js
import { square } from './math.js'
console.log('打印2的平方',square(2))

const app = document.getElementById('app')
app.innerHTML = 'hello world';
```
再建一个`index.js`依赖的模块文件`math.js`
```
//src/math.js
export function square(x) {
    return x * x;
}

export function cube(x) {
    return x * x * x;
}
```
新建打包需要的模板文件`index.html`
```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title><%= htmlWebpackPlugin.options.title %></title>
</head>
<body>
    <div id="app"></div>
</body>
</html>
```
最后配置我们需要的npm script 启动命令，package.json
```
//package.json
{
  "name": "webpack-4-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "clean-webpack-plugin": "^3.0.0",
    "html-webpack-plugin": "^3.2.0",
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.5",
    "webpack-dev-server": "^3.7.2",
    "webpack-merge": "^4.2.1"
  }
}
```
现在我们来直接打包，没有做`tree shaking`的优化，看下打包的大小；
```
E:\workCode\webpack-4-demo>npm run build

> webpack-4-demo@1.0.0 build E:\workCode\webpack-4-demo
> webpack --config webpack.prod.js

Hash: 59507293492d05eeeeff
Version: webpack 4.35.0
Time: 473ms
Built at: 2019-06-27 13:01:07
            Asset       Size  Chunks             Chunk Names
    app.bandle.js    4.7 KiB     app  [emitted]  app
app.bandle.js.map   3.97 KiB     app  [emitted]  app
       index.html  366 bytes          [emitted]
Entrypoint app = app.bandle.js app.bandle.js.map
[./src/index.js] 145 bytes {app} [built]
[./src/math.js] 104 bytes {app} [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [./node_modules/.3.2.0@html-webpack-plugin/lib/loader.js!./index.html] e:/wo
rkCode/webpack-4-demo/node_modules/.3.2.0@html-webpack-plugin/lib/loader.js!./in
dex.html 582 bytes {0} [built]
        + 3 hidden modules

E:\workCode\webpack-4-demo>
```
可以看出项目打包的大小是：`app.bandle.js    4.7 KiB`。我们`index.js`文件引入了`math.js`模块，只用到了`square`方法，并没有用到`cube`方法，查看打包之后的`app.bandle.js`，找到我们自己的代码：
```
...
/***/ "./src/index.js":
/*!**********************!*\
  !*** ./src/index.js ***!
  \**********************/
/*! no exports provided */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony import */ var _math_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./math.js */ "./src/math.js");

console.log('打印2的平方',Object(_math_js__WEBPACK_IMPORTED_MODULE_0__["square"])(2))

const app = document.getElementById('app')
app.innerHTML = 'hello world';

/***/ }),

/***/ "./src/math.js":
/*!*********************!*\
  !*** ./src/math.js ***!
  \*********************/
/*! exports provided: square, cube */
/***/ (function(module, __webpack_exports__, __webpack_require__) {

"use strict";
__webpack_require__.r(__webpack_exports__);
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "square", function() { return square; });
/* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "cube", function() { return cube; });
function square(x) {
    return x * x;
}

function cube(x) {
    return x * x * x;
}

/***/ })

/******/ });
//# sourceMappingURL=app.bandle.js.map
...
```
可以看到`cube`方法仍然被打包到了`app.bandle.js`里面。
现在我们使用`tree shaking`，看看看打包之后的文件大小。（`注意，也可以在命令行接口中使用webpack  --optimize-minimize 标记，来启用 TerserPlugin。`）
在`webpack 4`版本中，直接对生产环境配置`mode:'production'`，即可启用`tree shaking`，并将代码压缩，
然后在`package.json`文件加上`"sideEffects": false`，（`注意，所有导入文件都会受到 tree shaking 的影响。这意味着，如果在项目中使用类似 css-loader 并 import 一个 CSS 文件，则需要将其添加到 side effect 列表中，以免在生产模式中无意中将它删除：`）用来提示` webpack compiler `找出哪些代码是未被引用的，然后删除掉。
```
//webpack.prod.js
const merge = require('webpack-merge')
const common = require('./webpack.common.js')

module.exports = merge(common, {
    mode: 'production',
    devtool: 'source-map'
})
```
```
//package.json
{
  ...
  "sideEffects": false
  ...
}
```
继续运行`npm run build`，可以看到代码体积变小了`app.bandle.js   1.06 KiB`，
```
E:\workCode\webpack-4-demo>npm run build

> webpack-4-demo@1.0.0 build E:\workCode\webpack-4-demo
> webpack --config webpack.prod.js

Hash: d1508bf9df4f2ca17868
Version: webpack 4.35.0
Time: 662ms
Built at: 2019-06-27 13:41:06
            Asset       Size  Chunks             Chunk Names
    app.bandle.js   1.06 KiB       0  [emitted]  app
app.bandle.js.map   4.88 KiB       0  [emitted]  app
       index.html  366 bytes          [emitted]
Entrypoint app = app.bandle.js app.bandle.js.map
[0] ./src/index.js + 1 modules 249 bytes {0} [built]
    | ./src/index.js 145 bytes [built]
    | ./src/math.js 104 bytes [built]
Child html-webpack-plugin for "index.html":
     1 asset
    Entrypoint undefined = index.html
    [0] e:/workCode/webpack-4-demo/node_modules/.3.2.0@html-webpack-plugin/lib/l
oader.js!./index.html 582 bytes {0} [built]
        + 3 hidden modules

E:\workCode\webpack-4-demo>
```
再来看一下打包之后的代码`app.bandle.js`，现在整个 `bundle` 都已经被 `minify(压缩)` 和 `mangle(混淆破坏)`，但是如果仔细观察，则不会看到引入` cube `函数，但能看到 `square`函数的混淆破坏版本`{"use strict";var n;r.r(t),console.log("打印2的平方",(n=2)*n),document.getElementById("app").innerHTML="hello world"}`
```
!function(e){var t={};function r(n){if(t[n])return t[n].exports;var o=t[n]={i:n,l:!1,exports:{}};return e[n].call(o.exports,o,o.exports,r),o.l=!0,o.exports}r.m=e,r.c=t,r.d=function(e,t,n){r.o(e,t)||Object.defineProperty(e,t,{enumerable:!0,get:n})},r.r=function(e){"undefined"!=typeof Symbol&&Symbol.toStringTag&&Object.defineProperty(e,Symbol.toStringTag,{value:"Module"}),Object.defineProperty(e,"__esModule",{value:!0})},r.t=function(e,t){if(1&t&&(e=r(e)),8&t)return e;if(4&t&&"object"==typeof e&&e&&e.__esModule)return e;var n=Object.create(null);if(r.r(n),Object.defineProperty(n,"default",{enumerable:!0,value:e}),2&t&&"string"!=typeof e)for(var o in e)r.d(n,o,function(t){return e[t]}.bind(null,o));return n},r.n=function(e){var t=e&&e.__esModule?function(){return e.default}:function(){return e};return r.d(t,"a",t),t},r.o=function(e,t){return Object.prototype.hasOwnProperty.call(e,t)},r.p="",r(r.s=0)}([function(e,t,r){"use strict";var n;r.r(t),console.log("打印2的平方",(n=2)*n),document.getElementById("app").innerHTML="hello world"}]);
//# sourceMappingURL=app.bandle.js.map
```
更多详细的`tree shaking`介绍，大家可以参考官方文档。
参考：
https://webpack.js.org/guides/tree-shaking/#root
https://webpack.docschina.org/guides/production
https://webpack.docschina.org/guides/tree-shaking/
