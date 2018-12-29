---
title: rx.js初探
date: 2018-12-29 16:10:30
tags:
---
### RxJS
RxJS 是使用 Observables 的响应式编程的库，它使编写异步或基于回调的代码更容易。

### 简单demo
使用rx.js实现一个canvas画板
html
```html
<canvas></canvas>
```

```javaScript
   import {Observable} from 'rxjs/Rx' //引入rx Observable对象

   const canvas = document.querySelector('canvas')
   const ctx = canvas.getContext('2d') //获取canvas画笔
   ctx.beginPath()
   function drawEvent([first, sec]) {
	    ctx.moveTo(first.x, first.y)
	    ctx.lineTo(sec.x, sec.y)
	    ctx.stroke()
    }
   const moveEvent = Observable.fromEvent(canvas,'mousemove')
        .map(e =>{x:e.offsetX,y:e.offsetY})
        .bufferCount(2,1) //定义move流

    const downEvent = Observable.fromEvent(canvas,'mousedown').map(()=>'down') //将Observable 结果映射为'down'

    //这里我们可以将Observable想象成一个数组,将数组的map方法类似于这里的map方法

    const upEvent = Observable.fromEvent(canvas,'mouseup').map(()=>'up') //将Observable 结果映射为'up'

    const upAndDown = downEvent.merge(upEvent) //融合两个Observable

    upAndDown.switchMap(e=>e==='down'?moveEvent:Observable.empty()).subscribe(drawEvent)
    //subscribe: 注册处理程序的订阅引用
    //因为事件处理流可能是异步的,我们不必在等待事件执行成功后再继续下一步操作,而是订阅一个注册事件,事件执行后再通知给订阅者

```

### 总结
这只是一个简单的demo,用于展示RxJS的基本思想,一个面向过程的基本思想。
