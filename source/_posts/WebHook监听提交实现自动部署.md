---
title: 使用Github WebHooks监听提交实现hexo自动部署
date: 2019-09-02 17:14:36
tags:
---

### Webhooks
Webhooks允许在发生特定事件时通知外部服务。当指定的事件发生时，将向设置好的URL地址发送POST请求。

在GitHub中设置webhooks首先进入项目的设置页面,在左侧找到Webhooks选项,点击add进入新增设置

首先设置`Payload URL`,这是你的服务器地址,每当有提交动作就会像这个地址发送post请求

`Content type`可以设置表单或者json格式

`Secret`密钥,用于服务器验证请求是否有Github服务器发送

所有设置完毕后,就可以进行下一步

### 服务器监听
这里推荐使用`github-webhook-handler`这个库,可以很方便帮助我们完成服务端的配置
示例代码
```javascript
var http = require('http')
var createHandler = require('github-webhook-handler')
var handler = createHandler({path: '这里填写在github上配置的路径地址', secret: '这里填写上面项目上配置的密钥'})
const {exec} = require('child_process') // 执行本地命令

http.createServer(function(req, res) {
	handler(req, res, function(err) {
		res.statusCode = 404
		res.end('no such location')
	})
}).listen('配置的端口')

handler.on('error', function(err) {
	console.error('Error:', err.message)
})

handler.on('push', function(event) {
	console.log('Received a push event for %s to %s',
		event.payload.repository.name,
		event.payload.ref)
	exec('cd /www/wwwroot/default/blog && git pull', (err, stdout, stderr) => {
		if (err) {
			console.log(err)
		}
		else {
			console.log('updated success')
		}
	})
})
```

### 结束
这样,在服务端部署好这个小的程序后(注意开放相关端口),每次在本地执行```hexo g -d```命令后,服务端接收到钩子信息就会自动从master分支拉取最新的代码,实现自动更新
