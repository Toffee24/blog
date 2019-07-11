---
title: 收集常用的css代码
date: 2019-07-11 14:55:41
tags:
---

### 清除默认样式
#### reset
---
```css
html,body{height:100%;}
html,body,h1,h2,h3,h4,h5,h6,div,dl,dt,dd,ul,ol,li,p,blockquote,pre,hr,figure,table,caption,th,td,form,fieldset,legend,input,button,textarea,menu{margin:0;padding:0;}
header,footer,section,article,aside,nav,hgroup,address,figure,figcaption,menu,details{display:block;}
table{border-collapse:collapse;border-spacing:0;}
caption,th{text-align:left;font-weight:normal;}
html,body,fieldset,img,iframe,abbr{border:0;}
i,cite,em,var,address,dfn{font-style:normal;}
[hidefocus],summary{outline:0;}
li{list-style:none;}
h1,h2,h3,h4,h5,h6,small{font-size:100%;}
sup,sub{font-size:83%;}
pre,code,kbd,samp{font-family:inherit;}
q:before,q:after{content:none;}
textarea{overflow:auto;resize:none;}
label,summary{cursor:default;}
a,button{cursor:pointer;}
h1,h2,h3,h4,h5,h6,em,strong,b{font-weight:bold;}
del,ins,u,s,a,a:hover{text-decoration:none;}
body,textarea,input,button,select,keygen,legend{font:12px/1.14 Microsoft YaHei,arial,\5b8b\4f53;color:#333;outline:0;}
body{background:#fff;}
a,a:hover{color:#333;}
*{box-sizing: border-box;}
```
### 去除input默认填充的背景颜色
---
```css
input:-webkit-autofill {
  -webkit-box-shadow: 0 0 0px 1000px white inset;
}
```
### 清除input[type=number]的默认样式
---
```css
input[type=number] {
    -moz-appearance:textfield;
}
input[type=number]::-webkit-inner-spin-button,
input[type=number]::-webkit-outer-spin-button {
    -webkit-appearance: none;
    margin: 0;
}
```
### 清除移动端 a 标签等点击区域变色
---
```css
*{
  -webkit-tap-highlight-color: rgba(255, 255, 255, 0);
}
```
### 清除移动端 input 样式
---
```css
input{
    border: none;
    -moz-appearance:none;
    -webkit-appearance : none ; /*解决ios上按钮的圆角问题*/
    border-radius: 0; /*解决ios上输入框圆角问题*/
    outline:medium; /*去掉鼠标点击的默认黄色边框*/
    background-color: transparent;
}
```
### 避免ios滑动滚动条卡顿
---
```css
*{
  -webkit-overflow-scrolling : touch
}
```
### 滚动条样式
---
```css
.scroll-container {
 height: 250px;
 border: 1px solid #ddd;
 padding: 15px;
 overflow: auto;
 .row {
   margin: 0;
   line-height: 1.5;
 }

 &::-webkit-scrollbar {
   width: 8px;
   background: white;
 }
 &::-webkit-scrollbar-corner, /* 滚动条角落 */
 &::-webkit-scrollbar-thumb,
 &::-webkit-scrollbar-track {
   border-radius: 4px;
 }
 &::-webkit-scrollbar-corner,
 &::-webkit-scrollbar-track {
   /* 滚动条轨道 */
   background-color: rgba(180, 160, 120, 0.1);
   box-shadow: inset 0 0 1px rgba(180, 160, 120, 0.5);
 }
 &::-webkit-scrollbar-thumb {
   /* 滚动条手柄 */
   background-color: #00adb5;
 }
}
```
### 常用 midea媒体查询
---
```css
/* 横屏 */
@media screen and (orientation:landscape){
     
}
/* 竖屏 */
@media screen and (orientation:portrait){
     
}
/* 窗口宽度<960,设计宽度=768 */
@media screen and (max-width:959px){
     
}
/* 窗口宽度<768,设计宽度=640 */
@media screen and (max-width:767px){
     
}
/* 窗口宽度<640,设计宽度=480 */
@media screen and (max-width:639px){
     
}
/* 窗口宽度<480,设计宽度=320 */
@media screen and (max-width:479px){
     
}
/* 设备像素比为2 */
/* 常用于1px边框，还应规定 3dppx 的情况 */
@media (min-resolution: 2dppx) {

}
/* windows UI 贴靠 */
@media screen and (-ms-view-state:snapped){
     
}
/* 打印 */
@media print{
     
}
```
### 长文本折行
---
```css
.long-text{
  white-space: pre-line;
  word-wrap: break-word;
}
```
### 文字超出显示省略号
#### 单行文字
```css
/*注意宽度是必须的*/
.article-container {
  width: 500px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```
#### 多行文字
---
```css
.article-container {
  display: -webkit-box;
  word-break: break-all;
  -webkit-box-orient: vertical;
  -webkit-line-clamp: 4; //需要显示的行数
  overflow: hidden;
  text-overflow: ellipsis;
}
```
### 消除图片下方间隙
---
> 通过设置给 img 元素设置 vertical-align: top 消除行内元素的间隙（仅对 img 元素有用

```css
vertical-align: top;
```
### 图文居中
---
```html
<div class="container">
  <img src="../../public/images/bg1.jpg">
  <span>安能摧眉折腰事权贵，使我不得开心颜</span>
</div>
```
```css
.container{
  padding:15px 0;
  img{
    vertical-align: middle;
  }
}
```
