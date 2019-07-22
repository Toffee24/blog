---
title: vue transition实现原生页面跳转效果
date: 2019-08-21 11:00:11
tags:
---




### 代码
使用Vue `<transition>` 组件过渡

```
<transition :name="this.$store.routeAction">
     <router-view/>
</transition>
```
CSS
```
.push-enter-active,.push-leave-active
, .pop-enter-active,.pop-leave-active{
  transition: all 0.4s;
}

.push-leave-to{
transform: translate(-20%,0);
}

.push-enter {
  transform: translate(100%, 0);
}
.push-enter-active {
  z-index: 10;
}
.push-leave-active {
  z-index: 0;
}
.pop-leave-active {
  transform: translate(100%, 0);
  z-index: 11;
}

.pop-enter{
  transform: translate(-20%,0);
}

```
### 效果

![image](/blog/images/vue-transition.gif)