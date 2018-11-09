---
title: 使用async库并发爬取网易云音乐歌单
date: 2018-11-09 17:09:35
tags:
---

##### [async](https://github.com/caolan/async)是node的一个强大的异步第三方库,它包含许多功能方法,今天主要用其中的`mapLimit`方法来实现并行执行爬虫.`mapLimit(coll,limit, iteratee, callbackopt)`接收四个参数:
##### coll:是一个迭代器,代表要迭代的集合
##### limit:数字代表同时执行并行的限制
##### iteratee:迭代器方法,对于coll中的每一个item，迭代执行该异步函数。用(item, callback)调用，callback可选
##### callbackopt:所有iteratee 函数完成后或发生错误时触发的回调函数。用(err, results)调用。
```script
const puppeteer = require('puppeteer')
const cheerio = require('cheerio')
const mapLimit  = require('async/mapLimit')

const URL = 'https://music.163.com'
//标签集合
const type =['华语','欧美','日语','韩语','粤语','小语种']
async function start(type,callback){
	const browser = await puppeteer.launch({
		headless:true
	})
	const page = await browser.newPage()
	await page.goto(`${URL}/discover/playlist/?cat=${type}`)
	const $ = cheerio.load(await page.mainFrame().childFrames()[0].content())
	const list = $('#m-pl-container').find('li')
	let album = []
	Array.from(list).forEach(ele=>{
		let obj ={
			name:$(ele).find('.dec a').attr('title'),
			href:$(ele).find('.dec a').attr('href')
		}
		album.push(obj)
	})
	callback(null,album)
	await browser.close()
}

mapLimit(type,5,(type,callback)=>{
//设置并发数为 5
	start(type,callback)
},(err,result)=>{
	for(let index in result){
		console.log(result[index])
	}
})

```
result:
    ![](/images/album_list.png)

