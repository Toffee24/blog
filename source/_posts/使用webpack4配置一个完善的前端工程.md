---
title: 使用webpack4配置一个完整的react前端工程
date: 2019-06-13 15:18:00
tags:
---

## 一、基础配置
### 1、init
```
mkdir webpack-project
cd webpack-project
mkdir src
yarn init
```
### 2、安装
```javascript
yarn add webpack webpack-cli webpack-dev-server
touch webpack.config.js
// webpack.config.js
module.exports = {
    mode: "development",
    entry: ["./src/index.js"],
    output: {
    	path: path.join(__dirname,'dist'),
    	filename: "bundle.js"
    },
    module: {},
    plugins: {},
    devServer: {}
}
```
### 3、安装react
```
yarn add react react-dom react-router-dom
// 建立如下的文件目录，并编写安装react和react-router并编写react代码如下
|-src
│      index.js 主文件
├───pages
│      Count.jsx -- 实现了一个计数器的功能，点击按钮，会让数字增加，按钮会实时显示在页面上
│      Home.jsx -- 一个简单的文字展示
└───router
       index.js -- 路由配置文件，两个页面分别对应两个路由 count和 home
```
### 4、babel编译es6,jsx等
```
// @babel/core-babel核心模块    @babel/preset-env-编译ES6等 @babel/preset-react-转换JSX
yarn add babel-loader @babel/core @babel/preset-env @babel/plugin-transform-runtime @babel/preset-react
// @babel/plugin-transform-runtime: 避免 polyfill 污染全局变量，减小打包体积
// @babel/polyfill: ES6 内置方法和函数转化垫片
yarn add @babel/polyfill @babel/runtime
//module配置
{
    test: /\.jsx?$/,
    exclude: '/node_modules/',
    use: [{
        loader: 'babel-loader'
    }]
}
```
根目录新建.babelrc文件
```json
{
  "presets": ["@babel/preset-env","@babel/preset-react"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
```
<!-- more -->

### 5、按需引入babel-polyfill
一般地,安装完`@babel/polyfill`之后,在入口文件直接`import`,但这样会有导致引入不需要的语法垫片,导致打包出来的文件过大的问题
当配合`@babel/preset-env`一起使用时,更改`.babelrc`
`useBuiltIns`表示只打包项目使用到的
```
{
  "presets": ["@babel/preset-env",
              { "useBuiltIns": "usage" }, //entry 表示完整引入
              "@babel/preset-react"],
  "plugins": ["@babel/plugin-transform-runtime"]
}
 
```
如果没有用到`@babel/preset-env`时,可以在entry入口导入,但是官方建议还是不要这么做
```javascript
module.exports = {
	entry:['@babel/polyfill','./src/index.js']
}
```
### 6、插件CleanWebpackPlugin
每次打包都会在dist生成新的文件,所以需要这个插件在每次打包前删除上次打包好的文件
```javascript
yarn add CleanWebpackPlugin
import CleanWebpackPlugin from 'clean-webpack-plugin'
plugins :[
	new CleanWebpackPlugin()
	]
```
### 7、使用HtmlWebpackPlugin
现在我们生成的都是js文件,但是我们还需要html,使用这个插件可以通过设置的模板html生成自动关联js路径的最终的html
```javascript
yarn add html-webpack-plugin
cd src
touch index.html
//webpack.config.js
const HtmlWebpackPlugin = require('html-webpack-plugin');
plugins: [
    new CleanWebpackPlugin(),
    new HtmlWebpackPlugin({
      filename: 'index.html', // 最终创建的文件名
      template: path.join(__dirname, 'src/template.html') // 指定模板路径
    })
  ]
```
### 8、使用dev-tool配置项进行source-map优化
webpack中devtool选项用来控制是否生成，以及如何生成 source map。简言之，source map就是帮助我们定位到错误信息位置的文件。正确的配置source map，能够提高开发效率，更快的定位到错误位置。
```javascript
//webpack.config.js
module.exports = {
	devtool:'cheap-module-eval-source-map',//线上环境
	devtool:'cheap-module-source-map'//测试环境
}
```
### 9、使用WebpackDevServer配置开发服务器
`webpack-dev-server`是在本地开发部署的一个小型的静态资源服务器,提供实时重载功能
```javascript
//webpack.config.js
module.exports = {
	devServer: {
    	hot:true,
    	inline:true,
    	contentBase:path.join(__dirname,'dist'),
    	host:'0.0.0.0',//方便局域网访问
    	port:8080,
    	historyApiFallback: true, // 该选项的作用所有的404都连接到index.html
        proxy: {
    		// 代理到后端的服务地址，会拦截所有以api开头的请求地址, 解决跨域问题
            "/api": "http://xxx.xxx.x.xx:xxxx"
    	}
    }
}
```
### 10、使用HotModuleReplacement热更新(HMR)
建立了开发环境本地服务器后，当修改内容后，网页会同步刷新，我们现在进入toCount页面
- 点击按钮，将数字加到一个不为0的数，比如加到6
- 然后你可以在代码中改变按钮的文字，随便改点东西，会发现，页面刷新后，数字重新变为0
``这显然不是我们想要的，想要的是，能不能把页面的状态保存了，也就是更改了代码后，页面还是保存了数字为6的状态``,也就是局部改变。这时就要用到`HotModuleReplacement`这个插件
```javascript
//webpack.config.js
module.exports = {
	devServer: {
        	hot:true,
        	inline:true,
        	contentBase:path.join(__dirname,'dist'),
        	host:'0.0.0.0',//方便局域网访问
        	port:8080,
        	historyApiFallback: true, // 该选项的作用所有的404都连接到index.html
            proxy: {
        		// 代理到后端的服务地址，会拦截所有以api开头的请求地址, 解决跨域问题
                "/api": "http://xxx.xxx.x.xx:xxxx"
        	}
        },
	plugins:[new webpack.HotModuleReplacementPlugin()]
}
```
### 11、react-hot-loader记录react页面留存状态state
`react-hot-loader`是解决在react开发中的页面留存状态刷新问题
```javascript
cnpm i react-hot-loader -D
 
// 在主文件里这样写
 
import React from "react";
import ReactDOM from "react-dom";
import { AppContainer } from "react-hot-loader";-------------------1、首先引入AppContainre
import { BrowserRouter } from "react-router-dom";
import Router from "./router";
 
/*初始化*/
renderWithHotReload(Router);-------------------2、初始化
 
/*热更新*/
if (module.hot) {-------------------3、热更新操作
  module.hot.accept("./router/index.js", () => {
    const Router = require("./router/index.js").default;
    renderWithHotReload(Router);
  });
}
 
 
function renderWithHotReload(Router) {-------------------4、定义渲染函数
  ReactDOM.render(
    <AppContainer>
      <BrowserRouter>
        <Router />
      </BrowserRouter>
    </AppContainer>,
    document.getElementById("app")
  );
}
```
### 12、编译css和scss/less
```
yarn add css-loader style-loader sass-loader node-sass / less-loader
 //webpack.config.js
{
  test: /\.scss$/,
    use: [
      "style-loader", // 创建style标签，并将css添加进去
      "css-loader", // 编译css
      "sass-loader" // 编译scss
    ]
}
```
### 13、使用PostCSS
``PostCSS``是一个使用JS插件转换css样式的工具,最常用到的就是`Autoprefixer`这个插件,但还有许多的插件比如`postcss-preset-env`,可以使用实验性的css特性,
`stylelint`css语法检测等
```javascript
yarn add postcss-loader
//webpack.config.js
{
    test: /\.scss$/,
        use: [
            "style-loader", // 创建style标签，并将css添加进去
            "css-loader", // 编译css
            "postcss-loader",
            "sass-loader" // 编译scss
        ]
}
```
### 14、处理图片
```javascript
yarn add file-loader url-loader
 
url-loader 解决css等文件中引入图片路径的问题
file-loader 当图片较小的时候会把图片BASE64编码，大于limit参数的时候还是使用file-loader 进行拷贝
//webpack.config.js
{
    test: /\.(png|jpg|jpeg|gif|svg)/,
    use: {
      loader: 'url-loader',
      options: {
        outputPath: 'images/', // 图片输出的路径
        limit: 10 * 1024
      }
    }
}
```
### 15、处理字体
```javascript
{
	test: /\.(eot|woff2?|ttf|svg)$/,
    use: [
      {
        loader: 'url-loader',
        options: {
          name: '[name]-[hash:5].min.[ext]',
          limit: 5000, // fonts file size <= 5KB, use 'base64'; else, output svg file
          publicPath: 'fonts/',
          outputPath: 'fonts/'
        }
      }
    ]
  }
```
## webpack优化
### 1、extension,alias对文件路径后缀名优化
- extension: 指定extension之后可以不用在require或是import的时候加文件扩展名,会依次尝试添加扩展名进行匹配
- alias: 配置别名可以加快webpack查找模块的速度
```javascript
 resolve: {
    extension: ["", ".js", ".jsx"],
    alias: {
      "@": path.join(__dirname, "src"),
      pages: path.join(__dirname, "src/pages"),
      router: path.join(__dirname, "src/router")
    }
  }
```
### 2、MiniCssExtractPlugin抽取css文件
如果不做配置，我们的css是直接打包进js里面的，我们希望能单独生成css文件。 因为单独生成css,css可以和js并行下载，提高页面加载效率
```javascript
yarn add mini-css-extract-plugin
//webpack.config.js
const MiniCssExtractPlugin = require("mini-css-extract-plugin");
 {
  {
    test: /\.scss$/,
    use: [
      // "style-loader", // b不再需要style-loader要已经分离处理
      MiniCssExtractPlugin.loader,
      "css-loader", // 编译css
      "postcss-loader",
      "sass-loader" // 编译scss
    ]
   },

  plugins: [
    new MiniCssExtractPlugin({
      filename: "[name].css",
      chunkFilename: "[id].css"
    })
  ]
}
```
### 3、代码分割按需加载、提取公共代码
为什么要实现按需加载？
我们现在看到，打包完后，所有页面只生成了一个bundle.js,当我们首屏加载的时候，就会很慢。因为他也下载了别的页面的js了,也就是说，执行完毕之前，页面是完全空白的。 如果每个页面单独打包自己的js，就可以在进入页面时候再加载自己的js，首屏加载就可以快很多
```
optimization: {
	splitChunks:{
		chunks:'all'
	}
}
```
### 4、文件压缩
webpack4只要在生产模式下，代码就会自动压缩
```
mode: 'production'
```
### 5、使用外部扩展(全局变量)
可以直接在全局使用定义的`$`变量
```
import Webpack from 'webpack'

externals: {
    '$': 'jquery' //npm包 //需要require('$')
  }
//或者
plugins: [new Webpack.ProvidePlugin({
     $: 'jquery', // npm
     jQuery: 'jQuery' // 本地Js文件
})]
```
### 6、指定环境,定义环境变量
```javascript
import Webpack from 'webpack'

{
	plugins: [
        new webpack.DefinePlugin({
          'process.env': {
            VUEP_BASE_URL: JSON.stringify('http://localhost:8080')
          }
        }),
    ]
}
```
### 7、CSS Tree Shaking
```
yarn add glob-all purify-css purifycss-webpack
 
const PurifyCSS = require('purifycss-webpack')
const glob = require('glob-all')
plugins:[
    // 清除无用 css
    new PurifyCSS({
      paths: glob.sync([
        // 要做 CSS Tree Shaking 的路径文件
        path.resolve(__dirname, './src/*.html'), // 请注意，我们同样需要对 html 文件进行 tree shaking
        path.resolve(__dirname, './src/*.js')
      ])
    })
]
```
### 8、js Tree Shaking
清除到代码中无用的js代码，只支持import方式引入，不支持commonjs的方式引入
``只要mode是production就会生效，develpoment的tree shaking是不生效的，因为webpack为了方便你的调试``
```
 optimization: {
    usedExports:true,
  }
```
