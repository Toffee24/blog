---
title: 几种移动端rem布局解决方案
date: 2018-11-21 11:09:05
tags:
---

### rem概念
rem是一种css单位,通常用于解决移动端各个手机显示适配问题,保证UI设计在不同手机,不同设备之间显示效果的一致性。它是相对于html根元素的字体大小而定,比如设定根元素字体大小font-size:16px,那么1rem = 16px

### viewport
viewport是专为手机浏览器设计的一个meta标签；　有些屏幕很小有智能手机，但分辩率却可以做得很大，比如某些手机的默认分辨率为：1920*1080，比许多电脑桌面的都还大，传统桌面网站直接放到手机上阅读时，界面就会显得非常小，阅读体验就很差，就样就需要一种将原始视图在手机上放大的机制，使用viewport标签可以解决这个问题，如
```javaScript
<meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, user-scalable=no">

initial-scale：初始缩放比例
maximum-scale：允许缩放的最大比例
minimum-scale：允许缩放的最小比例
user-scalable：是否允许手动缩放
```

### 阿里解决方案(1rem = 100px)
把下面这段已压缩过的原生JS放到HTML的 head 标签中即可（注:不要手动设置viewport，该方案自动帮你设置）
```
<script>!function(e){function t(a){if(i[a])return i[a].exports;var n=i[a]={exports:{},id:a,loaded:!1};return e[a].call(n.exports,n,n.exports,t),n.loaded=!0,n.exports}var i={};return t.m=e,t.c=i,t.p="",t(0)}([function(e,t){"use strict";Object.defineProperty(t,"__esModule",{value:!0});var i=window;t["default"]=i.flex=function(normal,e,t){var a=e||100,n=t||1,r=i.document,o=navigator.userAgent,d=o.match(/Android[\S\s]+AppleWebkit\/(\d{3})/i),l=o.match(/U3\/((\d+|\.){5,})/i),c=l&&parseInt(l[1].split(".").join(""),10)>=80,p=navigator.appVersion.match(/(iphone|ipad|ipod)/gi),s=i.devicePixelRatio||1;p||d&&d[1]>534||c||(s=1);var u=normal?1:1/s,m=r.querySelector('meta[name="viewport"]');m||(m=r.createElement("meta"),m.setAttribute("name","viewport"),r.head.appendChild(m)),m.setAttribute("content","width=device-width,user-scalable=no,initial-scale="+u+",maximum-scale="+u+",minimum-scale="+u),r.documentElement.style.fontSize=normal?"50px": a/2*s*n+"px"},e.exports=t["default"]}]); flex(false,100, 1);</script>
```

### 网易解决方案(1rem = 100px)
```javaScript
var win=window
win.resize = {};

    var timer = null;
    var rem = 12;
    var doc = win.document;
    var docEl = doc.documentElement;

    /**
     * 刷新页面REM值
     */
    function refreshRem() {
        var width = docEl.getBoundingClientRect().width;
        width = width > 768 ? 640 : width;
        rem = width / 7.5;
        docEl.style.fontSize = rem + 'px';
    }

    /**
     * 页面缩放或重载时刷新REM
     */
    win.addEventListener('resize', function () {
        clearTimeout(timer);
        timer = setTimeout(refreshRem, 300);
    }, false);
    win.addEventListener('pageshow', function (e) {
        if (e.persisted) {
            clearTimeout(timer);
            timer = setTimeout(refreshRem, 300);
        }
    }, false);

    refreshRem();
```

### 美团解决方案(1rem = 100px)
```javaScript
    (function () {
        function o() {
            document.documentElement.style.fontSize = (document.documentElement.clientWidth && document.documentElement.clientWidth < 768 ? document.documentElement.clientWidth : 768) / 7.5 + "px"
        }

        var e = null;
        window.addEventListener("resize", function () {
            clearTimeout(e), e = setTimeout(o, 300)
        }, !1), o()

        //3秒后检查如果html fontSize为0，手动触发计算fontSize。
        setTimeout(function () {
            if (parseInt(document.defaultView.getComputedStyle(document.documentElement).fontSize) == 0) {
                o();
            }
        }, 3000);
    })(window)
```

### 纯css方案
```css
html {
  width: 100%;
  max-width:768px;
  font-size:calc(100vw/7.5);
}
```
