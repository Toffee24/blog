---
title: 1px border边框实现
date: 2018-11-16 13:32:12
tags:
---

由于移动端手机像素密度的差异,1px宽度的边框在不同的DPI下展示会有差异,可以通过下面这个方法来解决
```css
<!--解决1px边框-->
<!--@ SASS语法-->
@mixin border-1px($color){
  position: relative;
  &:after{
    display: block;
    position: absolute;
    left: 0;
    bottom: 0;
    width: 100%;
    border-top: 1px solid $color;
    content: '';
  }
}

<!--然后设置 统一的类 缩放-->
@media (-webkit-min-device-pixel-ratio: 1.5),(min-device-pixel-ratio: 1.5) {
  .border-1px{
    &:after{
      -webkit-transform: scaleY(0.7);
      transform: scaleY(0.7);
    }
  }
}
@media (-webkit-min-device-pixel-ratio: 2),(min-device-pixel-ratio: 2) {
  .border-1px{
    &:after{
      -webkit-transform: scaleY(0.5);
      transform: scaleY(0.5);
    }
  }
}
```
<!-- more -->
