---
title: event-bus发布订阅实现
date: 2018-12-12 17:10:15
tags:
---

一个简单的实现了事件注册,订阅,注销的eventEmeitter

```javaScript
class EventEmeitter{
    constructor(){
        this._events = new Map()
    }
    emit(type,context,...args){
        const handler = this._events.get(type)
        if(typeof handler !== 'function'){
            throw Error(`[${type}] 未绑定事件`)
        }
        if(args.length >= 3){
            handler.apply(context,args)
        }else{
            handler.call(context,...args)
        }
    }
    addEventListener(type,fn){
        if(!this._events.get(type)){
            this._events.set(type,fn)
            console.log(`[${type}] 事件绑定`)
        }else{
            throw Error(`[${type}] 事件已绑定`)
        }
    }
    removeEventListener(type){
        this._events.delete(type)
        console.log(`[${type}] 事件移除`)
    }
}

const emitter = new EventEmeitter()
emitter.addEventListener('christmas',s=>{
    console.log(`Merry ${s}`)
})
emitter.emit('christmas',this,'Christmas')
emitter.removeEventListener('Christmas')

---
[christmas] 事件绑定
Merry Christmas
[Christmas] 事件移除
```
