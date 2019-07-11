---
title: gulp配置react开发环境
date: 2019-05-25 22:18:21
tags:
---


### gulp与webpack
gulp是对文件的处理,是Task Runner,节省了手动编译,手动打包这一步,但是单纯使用gulp没法去解决js中引入module的问题,可能会遇到项目中明明引入了babel语法转换,为什么浏览器会提示`require is not defined`报错,这是因为babel只解析语法,并不会去管js的模块引入。
而webpack解决了require的问题,根据不同的文件转换需求引入不同的loader,最后打包出符合相应规则的静态资源文件。

### Browserify-让浏览器加载Node模块
<img src="/blog/images/browserify.png" width="50%">

官方文档开头介绍:`require('modules') in the browser`

目前NPM的包大多都是通过CMD的方式打包的，除了特定的可以使用CMD模块加载器加载的模块，大部分nodejs模块无法直接使用到浏览器环境中。
Browserify是一个供浏览器环境使用的模块打包工具，像在node环境一样，也是通过require('modules')来组织模块之间的引用和依赖，既可以引用npm中的模块，也可以引用自己写的模块，然后打包成js文件，再在页面中通过`<script>`标签加载。

### Browserify.transform
指定转换模块,在打包过程中自动执行,在这里需要结合babel进行语法转换


### 详细配置
react语法解析主要用到babel的`@babel/preset-react`插件,此外还会使用到`browserSync`进行自动检测刷新,`less`预处理
```JavaScript
const gulp = require('gulp') //引入gulp
const less = require('gulp-less') //引入gulp-less处理less语法
const LessAutoprefix = require('less-plugin-autoprefix') //css兼容前缀
const autoprefix = new LessAutoprefix({
    browsers: ['last 2 versions']
})
const del = require('del') //删除文件模块,在每次gulp任务前执行
const babel = require('gulp-babel') //babel
const browserSync = require('browser-sync').create() //自动刷新
const reload = browserSync.reload //reload示例,用于浏览器刷新
const browserify = require('browserify') //模块处理
const source = require('vinyl-source-stream') //将browerify流转换成gulp文件流
const babelify = require('babelify') //用于browserify的babel编译器

//主任务
function main(cb) {
    (async () => {
        await del.sync('dist')
        //less
        gulp.src('./src/index.less').pipe(less({
            plugins: [autoprefix]
        })).pipe(gulp.dest('./dist'))
        //js
        browserify({
            entries:['./src/index.js']
        })
        .transform(babelify,{
            presets: ['@babel/env','@babel/preset-react'],
            plugins: ["@babel/plugin-transform-runtime"]
        })
        .bundle() //打包
        .pipe(source('index.js')).pipe(gulp.dest('./dist'))
        gulp.src('./src/index.html').pipe(gulp.dest('./dist'))
        cb()
    })()

}

//开发环境
function devServer(cb){
    browserSync.init({
        server: {
            baseDir: './dist'
        }
    })
    gulp.watch("./src/**", gulp.series(lessChange,jsChange,htmlChange))
    cb()
}

//检测html变化
function htmlChange(){
    return gulp.src('./src/index.html').pipe(gulp.dest('./dist')).pipe(reload({stream:true}))
}

//检测样式文件
function lessChange() {
    return gulp.src('./src/index.less').pipe(less({
        plugins: [autoprefix]
    })).pipe(gulp.dest('./dist')).pipe(reload({stream:true}))
}

//检测js文件
function jsChange(cb){
    browserify({
        entries:['./src/index.js']
    })
    .transform(babelify,{
        presets: ['@babel/env','@babel/preset-react'],
        plugins: ["@babel/plugin-transform-runtime"]
    })
    .bundle()
    .pipe(source('index.js')).pipe(gulp.dest('./dist')).pipe(reload({stream:true}))
    cb()
}

//default默认任务,用于package.json执行脚本gulp任务
gulp.task('default', gulp.series(main,devServer))
```
