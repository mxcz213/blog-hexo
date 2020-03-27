---
title: webpack-4-搭建-React-架构：-antd-中后台项目架构（一）
---
前言：最近工作需要，有三个项目，需要做三个中后台管理页面，除了页面不一样，其他的内容组织都一样，所以搭建这么一个通用的项目架构来应用到这个三个项目中，以下是笔记总结。

前端工程师要有自己架构项目的能力，`create-react-app` 生成的是一个比较通用的项目结构，通常里面很多内容我们是用不到的，而且要改起来里面的 `webpack` 配置也更是繁琐，这就需要我们自己搭建一个适合自己项目的框架，然后配置必要的优化方案，这样能对这个项目做到了如指掌。也能锻炼我们的架构能力，组织代码的能力。

下面我们基于 `React` 框架架构一个自己的项目，亲手配置开发环境 `webapck 4` 配置（本地跑起服务）；
生产环境的 `webpack` （网络优化，缓存，压缩代码等）配置，基于 `webpack 4` 做相应打包构建优化。

系统环境要求：
基于`windows`系统，node版本是`v8.9.1`，npm版本是`v5.5.1`，webpack的版本是`v4.35.0`，安装模块使用淘宝镜像，这样使安装更快速，`$ npm install -g cnpm --registry=https://registry.npm.taobao.org`

首先应该是要先考虑项目如何架构，模块如何划分，就近维护原则，以及单一职责原则，工具函数怎么规划等，我们可以借鉴`vue-cli`的架构模式，其中有很多我们可以学习效仿的。

我们自己写的模块都放到 `src` 下面，`index.html` 作为 `html` 的模板，`webpack` 的配置直接放在根目录，这样更清晰，`dist` 目录用来放打包的项目代码，这个在 `webpack` 里面配置好，不需要手动建，每次 `build` 完之后清除目录重新打包，`mockData` 用来做数据模拟，`assets` 目录来放置静态图片和样式。

目录建好之后，第一步先把项目跑起来，接下来做具体的配置；
####安装依赖

* 首先初始化项目
```
npm init -y
```
* 基于webpack，所以我们要安装webpack相关：`webpack webpack-cli webpack-dev-server webpack-merge`，因为我们是开发和生产环境配置分离，需要`webpack-merge`这个工具来合并配置；
```
cnpm install webpack webpack-cli webpack-dev-server webpack-merge --save-dev
```
* 安装处理`css`文件的模块，`css-loader` 负责将 `js` 中 `import` 的 `css` 文件提取出来，`style-loader` 负责将 `css` 插入到 `head` 的 `style`v标签内，`autoprefixer` 浏览器前缀自动补全，`less-loader` 处理 `less` 文件，`postcss-loader` 将 `css` 解析为 `css`，这个过程中可以使用很多插件来继续处理 `css` ，比如`autoprefixer`， `cssnano`，`cssnext`等；
```
cnpm install css-loader style-loader postcss-loader autoprefixer less less-loader --save-dev
```
* 安装 `babel` 转换 `javascript`  语法，转换es6，es7的新方法，安装`@babel/core babel-loader @babel/preset-env @babel/preset-stage-2 @babel/runtime @babel/plugin-transform-runtime @babel/plugin-transform-regenerator babel-polyfill`
![](https://upload-images.jianshu.io/upload_images/5541401-469960a7950e912f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
如果配置好运行 `webpack`，出现这错误，说明 `babel-core babel-loader` 版本不对应，安装对应版本即可。

```
cnpm install @babel/core babel-loader @babel/preset-env @babel/preset-stage-2 @babel/runtime @babel/plugin-transform-runtime @babel/plugin-transform-regenerator babel-polyfill --save-dev
```
* 安装处理静态资源 `file-loader url-loader`
```
cnpm install file-loader url-loader --save-dev
```
* 安装根据 `html` 模板打包动态生成 `html` 文件的插件 `html-webpack-plugin`
```
cnpm install html-webpack-plugin --save-dev
```
* 安装提取 `css` 到独立样式文件的插件 `extract-text-webpack-plugin`，安装提示说这个插件在 `webpack 4` 版本已弃用，采用异步加载，性能更好的的 `mini-css-extract-plugin` 代替， `mini-css-extract-plugin` 支持按需加载 `css` 和 `sourceMap`，只能用在 `webpack4` 中，由于这个插件不支持 `HMR`，所以只在生产环境上配置。
![](https://upload-images.jianshu.io/upload_images/5541401-9cbf6658855a706a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
```
//cnpm install extract-text-webpack-plugin --save-dev
cnpm install mini-css-extract-plugin --save-dev
```
* 安装用来生产环境优化/最小化 `css` 代码的插件 `optimize-css-assets-webpack-plugin`：
```
cnpm install optimize-css-assets-webpack-plugin --save--dev
```
* 安装压缩 `js` 的插件 `uglifyjs-webpack-plugin`
```
cnpm install uglifyjs-webpack-plugin --save--dev
```
* 安装用于清除 `dist` 构建文件夹的插件，`clean-webpack-plugin`
```
cnpm install clean-webpack-plugin --save-dev
```
* 继续安装 `React` 相关配置：`react react-dom react-router` 
```
cnpm install react react-dom react-router --save-dev
```
* 最后安装 `React` 的组件库` antd`：`antd babel-plugin-import`，`babel-plugin-import` 用来按需引入 `antd` 的组件；
####配置  `.babelrc` 文件
```
//.babelrc
{
    "presets": [
        [
            "@babel/preset-env",
            {
                "modules": false,
                "targets": {
                    "browsers": ["> 1%", "last 2 versions", "not ie <=8"]
                }
            }
        ],
        "@babel/preset-react"
    ],
    "plugins": [
        [
            "import",
            {
                "libraryName": "antd",
                "style": true
            }
        ],
        "@babel/transform-runtime", "@babel/plugin-transform-regenerator"
    ]
}
```
####配置 `postcss.config.js` 文件
```
module.exports = {
    plugins: [
        require('autoprefixer')(),
    ]
}
```
####配置 `package.json` 中 `script` 运行、打包命令
```
{
  "name": "webpack-4-demo",
  "version": "1.0.0",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "start": "webpack-dev-server --open --config webpack.dev.js",
    "dev": "webpack-dev-server --open --config webpack.dev.js",
    "build": "webpack --config webpack.prod.js"
  },
  "author": "",
  "license": "ISC",
  "devDependencies": {
    "@babel/core": "^7.0.0",
    "@babel/plugin-transform-regenerator": "^7.4.5",
    "@babel/plugin-transform-runtime": "^7.5.0",
    "@babel/preset-env": "^7.5.0",
    "@babel/preset-react": "^7.0.0",
    "@babel/preset-stage-2": "^7.0.0",
    "@babel/runtime": "^7.5.0",
    "autoprefixer": "^9.6.0",
    "babel-loader": "^8.0.6",
    "babel-plugin-import": "^1.12.0",
    "babel-polyfill": "^6.26.0",
    "clean-webpack-plugin": "^3.0.0",
    "css-loader": "^3.0.0",
    "file-loader": "^4.0.0",
    "html-webpack-plugin": "^3.2.0",
    "less": "^3.9.0",
    "less-loader": "^5.0.0",
    "mini-css-extract-plugin": "^0.7.0",
    "optimize-css-assets-webpack-plugin": "^5.0.3",
    "postcss-cssnext": "^3.1.0",
    "postcss-import": "^12.0.1",
    "postcss-loader": "^3.0.0",
    "react": "^16.8.6",
    "react-dom": "^16.8.6",
    "react-router": "^5.0.1",
    "style-loader": "^0.23.1",
    "uglifyjs-webpack-plugin": "^2.1.3",
    "url-loader": "^2.0.1",
    "webpack": "^4.35.0",
    "webpack-cli": "^3.3.5",
    "webpack-dev-server": "^3.7.2",
    "webpack-merge": "^4.2.1"
  },
  "sideEffects": false,
  "dependencies": {
    "antd": "^3.20.0"
  },
  "theme": "./src/theme.js"
}
```
注意：`theme` 字段是 `antd` 主题相关的配置
`src/theme.js` 文件
```
/*
 *定制antd 样式
*/
module.exports = {
    'text-color':'#576077',
    'font-size-base':'14px', //默认字体大小
    'layout-header-background': '#0c8cee', //头部的颜色
    'layout-sider-background': '#2f3847', //边栏的颜色
    'layout-body-background': '#e4f3ff', //页面的背景颜色
    'layout-header-height':'50px', //头部的高
    'layout-header-padding':0, //头部的padding
    'card-head-color':'#333',
    'card-head-background':'#f7f9f9', //卡片的头部背景颜色
    'primary-color':'#3eacff', //确认的颜色
    'primary-5':'#0c8cee' , //确认的hover颜色
    'tabs-card-head-background':'#fff', //tab的headerColor
    'tabs-title-font-size':'14px',
    'tooltip-max-width':'500px',
    'tooltip-bg':'#efefef',
    'tooltip-color':'#000',
    'tooltip-arrow-width':'10px',
    'table-header-bg':'#e4f3ff',  //表格头部的背景颜色
    'border-radius-base':0,
    'border-radius-sm':0
}
```
####开发环境和生产环境公共配置 `webpack.common.js`
```
const path = require('path')
const HtmlWebpackPlugin = require('html-webpack-plugin')
const theme = require('./src/theme')

module.exports = {
    entry: {
        app: './src/main.js',
    },
    plugins: [
        new HtmlWebpackPlugin({
            title: 'webpack 4 production',
            template: 'index.html',
            filename: 'index.html'
        }),
    ],
    output: {
        filename: 'js/[name].[hash:8].js',
        path: path.resolve(__dirname, 'dist')
    },
    resolve: {
        extensions: ['.js', '.css', '.less']
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                use: 'babel-loader',
                exclude: path.resolve(__dirname, 'node_modules'),
                include:path.resolve(__dirname, 'src')
            },
            {
                test: /\.(woff2?|eot|ttf|otf|svg)(\?.*)?$/,
                loader: 'url-loader',
                include: /fonts?/,
                options: {
                    limit: 1024,
                    name: 'fonts/[name].[hash:7].[ext]'
                }
            },
            {
                test: /\.(png|jpe?g|gif|svg)(\?.*)?$/,
                loader: 'url-loader',
                exclude: /fonts?/,
                options: {
                    limit: 4096,                                
                    name: 'images/[name].[hash:7].[ext]'                        
                }
            },
            {
                test: /\.(mp4|webm|ogg|mp3|wav|flac|aac)(\?.*)?$/,
                loader: 'url-loader',
                options: {
                    limit: 4096,
                    name: 'media/[name].[hash:7].[ext]'
                }
            },
        ]
    }
}
```
####开发环境配置`webpack.dev.js`
```
const path = require('path')
const merge = require('webpack-merge')
const common = require('./webpack.common.js')
const webpack = require('webpack')
const theme = require('./src/theme')

module.exports = merge(common, {
    mode: 'development',
    devtool: 'inline-source-map',
    devServer: {
        contentBase: path.resolve(__dirname, 'dist'),
        hot: true,
        port: 9000,
        overlay: {
            warnings: false,
            errors: true
        }
    },
    plugins: [
        new webpack.HotModuleReplacementPlugin(),   //启用HMR,配合server的hot
    ],
    module: {
        rules: [
            {
                test:/\.css$/,
                use:[
                  'style-loader',
                  'css-loader',
                  'postcss-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                  'style-loader',
                  'css-loader',
                  'postcss-loader',
                  {
                    loader: 'less-loader',
                    options: {'modifyVars':theme,'javascriptEnabled': true}
                  }
                ]
            }
        ]
    }
})
```
####生产环境优化配置`webpack.prod.js`
```
const merge = require('webpack-merge')
const common = require('./webpack.common.js')
const { CleanWebpackPlugin } = require('clean-webpack-plugin')
const MiniCssExtractPlugin = require('mini-css-extract-plugin')
const UglifyJsPlugin = require('uglifyjs-webpack-plugin')
const OptimizeCSSAssetsPlugin = require('optimize-css-assets-webpack-plugin')
const path = require('path')
const theme = require('./src/theme')

module.exports = merge(common, {
    mode: 'production',
    devtool: 'source-map',
    plugins: [
        new CleanWebpackPlugin(),
        new MiniCssExtractPlugin({
            filename: 'css/[name].[chunkhash:8].css',
           // chunkFilename: 'css/[name]-[id].[chunkhash:8].css',
        }),
    ],
    optimization: {
        minimizer: [
            new UglifyJsPlugin({
                cache: true,
                parallel: false,
                sourceMap: false,
                uglifyOptions:{
                    output: {
                        ascii_only: true
                    }
                }
            }),
            new OptimizeCSSAssetsPlugin({})
        ],
        // splitChunks:{
        //     minChunks: 2,
        //     cacheGroups: {
        //         vendor: {
        //             name: 'vendor',
        //             chunks: 'initial',
        //             test: /[\\/]node_modules[\\/]/
        //         }
        //     }
        // }
    },
    module: {
        rules: [
            {
                test: /\.css$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                        options: {

                        }
                    },
                    'css-loader',
                    'postcss-loader'
                ]
            },
            {
                test: /\.less$/,
                use: [
                    {
                        loader: MiniCssExtractPlugin.loader,
                        options: {

                        }
                    },
                    'css-loader',
                    'postcss-loader',
                    {
                        loader: 'less-loader',
                        options: {'modifyVars':theme,'javascriptEnabled': true}
                    }
                ]
            }
        ]
    },
    performance: {
        hints: false
    }
})
```
####运行项目 打包构建
```
npm run dev
npm run build
```
最后：在项目开发过程中，可以逐渐完善项目的架构。
参考：
https://webpack.docschina.org/guides/
https://www.babeljs.cn/docs/babel-preset-stage-2
https://ant.design/docs/react/customize-theme-cn





