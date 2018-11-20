---
title: gulp配置示例
date: 2018-11-20 20:11:01
tags:
---
<img src="/blog/images/gulp.jpg" width="50%" height="50%">
### Gulp是什么
[Gulp](https://www.gulpjs.com.cn/docs/getting-started/)是一个基于流的构建工具，可以自动执行指定的任务，简洁且高效

### Gulp能做什么
- 开发环境下，想要能够按模块组织代码，监听实时变化
- css/js预编译，postcss等方案，浏览器前缀自动补全等
- 条件输出不同的网页，比如app页面和mobile页面
- 线上环境下，我想要合并、压缩 html/css/javascritp/图片，减少网络请求，同时降低网络负担
- 等等...

### gulp配置示例以及注释
```JavaScript
/******************************************************
*  gulp 组件 及 gulp task
******************************************************/

const gulp = require('gulp');  //引入gulp
const source = require('vinyl-source-stream');  //加载vinyl文件流处理
const browserify = require('browserify');  //加载预编译browserSync
const browserSync = require('browser-sync'); //实时更新插件
const buffer = require('vinyl-buffer');
const babel = require('babelify'); //babel语法转化
const reload = browserSync.reload;
const sourcemaps = require('gulp-sourcemaps');  //生成sourcemap文件
const del = require('del');  //引入删除模块
const argv = require('yargs').argv;  // 引入参数 根据参数的不同 执行相应的任务 s 生产环境  p  测试环境  d 开发环境
const __ = require('gulp-load-plugins')();  //可以不用引入 直接使用已经在package.json里的插件

//先预定好各个元素的路径
const domSrc = {
    //js 路径
    jsSrc: 'app/js/**/*.js',
    //js 库路径
    jsLibsSrc: 'app/js/libs/**/*.js',
    //jquery 文件
    jquerySrc:'app/js/libs/jquery/*.js',
    //config js路径
    configSrc: 'app/js/config/*.js',
    //index路径
    indexHtmlSrc: 'app/index.html',
    //页面路径
    htmlSrc: 'app/**/*.html',
    //css路径
    cssSrc: ['app/assets/**/*.css', 'app/assets/scss/app.scss'],
    //监听css路径
    watchCssSrc: 'app/assets/**/*.{css,scss}',
    //监听css路径
    //fonts路径 排除css文件
    fontSrc: ['app/assets/fonts/*', '!app/assets/fonts/*.css'],
    //image路径
    imgSrc: 'app/assets/images/**/*',
    //res路径
    resSrc: 'app/assets/res/**/*',
    //dist 下的 JS
    distJsSrc: 'dist/js',
    //dist 下的 html
    distHtmlSrc: 'dist/html',
    //dist 下的 tpl
    distTplSrc: 'dist/tpl',
    //dist 下的css
    distCssSrc: 'dist/css',
    //dist 下的 fonts
    distFontsSrc: 'dist/fonts'
}
//js 库文件 组文件 筛选
const libsfilter = __.filter(domSrc.jsLibsSrc, {restore: true, passthrough: false});
//js 业务 文件 筛选
const servicefilter = __.filter([domSrc.jsSrc, '!' + domSrc.jsLibsSrc], {restore: true, passthrough: false});
//html index.html 筛选
const indexHtmlfilter = __.filter(domSrc.indexHtmlSrc, {restore: true, passthrough: false});

//配置config
gulp.task('js:config', ()=> {
    var configSource = 'app/js/config/dev.js';
    if (argv.s) {
        //测试环境
        configSource = 'app/js/config/staging.js';
    } else if (argv.p) {
        //生产环境
        configSource = 'app/js/config/prod.js';
    }
    return gulp.src(configSource)
        .pipe(__.rename('config.js'))
        .pipe(gulp.dest('app/js/config'));
});

//ES6 to ES5
gulp.task('js:es6-2-es5', ()=> {
    return browserify({entries: 'app/js/myApp.js'})
        .transform(babel) //{presets: ["es2015"]}
        .bundle()
        //.pipe(__.sourcemaps.init())
        .pipe(source('myApp.js'))
        .pipe(__.if(argv.p, __.streamify(__.uglify())))
        .pipe(__.rename({suffix: '.min'}))
        //.pipe(__.sourcemaps.write('./'))
        .pipe(gulp.dest('dist/js'));
});

//JS 拷贝 libs ,components 文件夹
gulp.task('js:copy-libs', ()=> {
    return gulp.src([domSrc.jsLibsSrc,'!'+domSrc.jquerySrc])
        //.pipe(__.sourcemaps.init())
        .pipe(__.flatten()) //移除目录结构
        .pipe(__.concat("applibs.js")) //合并
        .pipe(__.uglify()) //压缩
        .pipe(__.rename({suffix: '.min'})) //重命名
        //.pipe(__.sourcemaps.write('./'))
        .pipe(gulp.dest('dist/js/libs'));
});

//解析HTML
gulp.task('html', ()=> {
    return gulp.src(domSrc.htmlSrc)
        .pipe(gulp.dest('dist/'))
});

//复制 fonts
gulp.task('fonts', ()=> {
    return gulp.src(domSrc.fontSrc)
        .pipe(gulp.dest('dist/css'));
});

//复制 图片
gulp.task('image', ()=> {
    return gulp.src(domSrc.imgSrc)
        .pipe(gulp.dest('dist/imgs'));
});

//复制 res
gulp.task('res', ()=> {
    return gulp.src(domSrc.resSrc)
        .pipe(gulp.dest('dist/res'));
});

//解析并合并压缩css
gulp.task('css', ()=> {
    return gulp.src(domSrc.cssSrc)
        .pipe(__.if('*.scss', __.sass()))
        .pipe(__.concat('app.css'))
        .pipe( __.minifyCss())
        .pipe(__.rename((path)=> {
            path.basename += '.min';
        }))
        .pipe(gulp.dest('dist/css'));
});

//清除dist下的JS文件
gulp.task('clean:js', ()=> {
    return del([domSrc.distJsSrc]);
});

//清除dist 下的html文件
gulp.task('clean:html', ()=> {
    return del([domSrc.distHtmlSrc, domSrc.distTplSrc]);
});

//清除dist下的 css 文件
gulp.task('clean:css', ()=> {
    return del([domSrc.distCssSrc]);
});

//清除dist 下的 fonts文件
gulp.task('clean:fonts', ()=> {
    return del([domSrc.distFontsSrc])
});

//本地服务器
gulp.task('server', ()=> {
    browserSync({
        // tunnel: true,
        open: false,
        port: 9996,
        server: {
            baseDir: ['dist'],
            index: ['./html/index.html', 'index.html']
        }
    });

    //js
    gulp.watch([domSrc.jsSrc,"!"+domSrc.configSrc], gulp.series('js', reload));
    //css
    gulp.watch(domSrc.watchCssSrc, gulp.series('css', reload));
    //html
    gulp.watch(domSrc.htmlSrc, gulp.series('html', reload));
    //image
    gulp.watch(domSrc.imgSrc, gulp.series('image', reload));
});

gulp.task('clean', gulp.parallel('clean:js', 'clean:html', 'clean:css', 'clean:fonts'));

gulp.task('js', gulp.series('clean:js', 'js:config', 'js:es6-2-es5', 'js:copy-libs'));

gulp.task('copy', gulp.parallel('html', 'fonts', 'image', 'res'));

gulp.task('default', gulp.series('clean', 'copy', 'js', 'css', 'server'));

gulp.task('build', gulp.series('clean', 'copy', 'js', 'css'));
```

