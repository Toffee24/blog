---
title: chrome快捷键太反人类?不想安装额外插件?那就用TamperMonkey写个脚本
date: 2019-07-19 17:15:07
tags:
---

### Chrome快捷键
对于一天用到将近百余次的`ctrl`+`w`的我来说,一直觉得chrome原本的`ctrl`+`w`的按键有点别扭,终于在今天忍不住了,开始寻找如何去修改chrome默认的快捷键。在设置里找了一圈,发现没有,无奈google了一下,发现有这么几种办法
- 安装chrome插件 如Shortkeys等
- 修改chrome系统文件
我综合了一下,发现我只需要实现`alt` + `w`关闭浏览器TAB的功能,这么折腾似乎不太经济,突然想到我有安装`TamperMonkey`油猴插件,于是便想到自己动手写个js脚本完事

### TamperMonkey
TamperMonkey是一个浏览器插件,它是一个脚本管理器,通过[脚本市场](https://greasyfork.org/zh-CN)安装第三方js脚本,可以实现非常多的拓展功能,可以就不展开了
我们新建一个脚本文件,注意脚本文件开头 `match`选项表示脚本应用的范围,这里我们应用所有网站
```
// @match        http://*/*
// @match        https://*/*
```

### 编写脚本
好了,下面就开始编写我们的脚本,TamperMonkey脚本文件实质就是输出个js IFFE(立即执行函数),
脚本内容都编写在里面,我们只需要全局监听`onkeydown`事件,判断是否同时按下`alt` + `w`,然后执行`window.close()`关闭页面就可以了
```javascript
(function() {
    window.addEventListener('keydown',function(event){
       if (event.altKey && event.keyCode == 87) {
           window.location.href = "about:blank"
          window.close()
    }
    })
})()
```
