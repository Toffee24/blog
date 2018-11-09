---
title: 使用Puppeteer爬虫
date: 2018-11-08 14:09:35
---

##### [Puppeteer](https://github.com/GoogleChrome/puppeteer)是 GoogleChrome团队官方的无界面（Headless）Chrome工具，它是一个Node库，提供了一个高级的API来控制[DevTool](https://chromedevtools.github.io/devtools-protocol/)协议上的无头版 Chrome 。也可以配置为使用完整（非无头）的 Chrome。

```javascript
const puppeteer = require('puppeteer')
const cheerio = require('cheerio')
const fs = require('fs')
const assert = require('assert')

async function start() {
	const browser = await puppeteer.launch()
	const page = await browser.newPage()
	//过滤请求资源
//	await page.setRequestInterception(true)
//	page.on('request', interceptedRequest => {
//		if (interceptedRequest.url().endsWith('.png') ||
//			interceptedRequest.url().endsWith('.jpg') ||
//			interceptedRequest.url().endsWith('.gif'))
//			interceptedRequest.abort()
//		else
//			interceptedRequest.continue()
//	})
	await page.goto('http://jandan.net/ooxx')
	const $ = cheerio.load(await page.content())
	const $img = $('img')
	for (let index in $img) {
		try {
			const content = await getResourceContent(page, ($img[index].attribs.src))
			const contentBuffer = Buffer.from(content, 'base64')
			const fileName = $img[index].attribs.src.match(
				/^http(s)?:\/\/(.+)\/(.+)\/(.+\..+)/)[4]
			fs.writeFileSync(fileName, contentBuffer, 'base64')
		}
		catch (e) {}
	}
	await browser.close()
}

//获得资源树
async function getResourceTree(page) {
	var resource = await page._client.send('Page.getResourceTree')
	return resource.frameTree
}

//根据url frameId查找资源实例
async function getResourceContent(page, url) {
	const {content, base64Encoded} = await page._client.send(
		'Page.getResourceContent',
		{frameId: String(page.mainFrame()._id), url},
	)
	assert.strictEqual(base64Encoded, true)
	return content
}

start()

```
