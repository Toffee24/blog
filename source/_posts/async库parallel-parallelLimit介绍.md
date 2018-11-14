---
title: async库parallel/parallelLimit介绍
date: 2018-11-13 14:54:38
tags:
---
### parallel
async.js是一个功能强大的node异步流程控制库,parallel是async中用来执行并行任务流程的一个方法,parallel(tasks,callbackopt)接收两个参数:

|Name|Type|Description
|-|-|-|
|tasks| Array/Iterable/Object | 一个用于遍历的数据类型
|callback| Function | (可选)返回一个数组或者对象,参数(err,result)

```javaScript
const parallel = require('async/parallel')
const time = new Date().getTime()

parallel([(callback)=>{
	setTimeout(()=>{
		callback(null,'one')
	},1200)
},(callback)=>{
	setTimeout(()=>{
		callback(null,'two')
	},1000)
}],(err,results)=>{
	console.log('time cost:',new Date().getTime()-time)
	console.log(results)
})
```
执行结果:
```
time cost: 1201
[ 'one', 'two' ]
```
parallel会等到所有函数返回结果后再返回最终的结果数组,并且按照顺序返回,不会因为第二个函数setTimeout的时间比第一个短就先返回

### parallelLimit
parallelLimit基本功能和parallel是一样的,不过它新增一个参数,用来控制同时执行的函数数量

|Name|Type|Description
|-|-|-|
|tasks| Array/Iterable/Object | 一个用于遍历的数据类型
|limit| number | 异步操作的最大数量
|callback| Function | (可选)返回一个数组或者对象,参数(err,result)

```javaScript
const parallelLimit = require('async/parallelLimit')
const time = new Date().getTime()

parallelLimit([(callback)=>{
	setTimeout(()=>{
		callback(null,'one')
	},1200)
},(callback)=>{
	setTimeout(()=>{
		callback(null,'two')
	},1000)
}],1,(err,results)=>{
	console.log('time cost:',new Date().getTime()-time)
	console.log(results)
})
```
运行结果
```
time cost: 2205
["one", "two"]
```
因为设置的limit是1,所以运行时间是所有运行时间之和
