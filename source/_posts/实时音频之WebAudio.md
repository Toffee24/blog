---
title: 实时音频之MediaDevices与AudioContext
date: 2021-03-10 17:26:44
tags:
---

### 前言
由于项目需要对外封装一个实时语音识别调用的JS-SDK，调研了相关实现方法，发现`mediaDevices`API可以实现实时音频传输识别结果的场景。

### 兼容性
<strong>不兼容</strong>ie 浏览器，其他浏览器兼容性具体文档可参考 [https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices)
<!-- more -->

### [navigator.mediaDevices.getUserMedia](https://developer.mozilla.org/zh-CN/docs/Web/API/MediaDevices/getUserMedia)

获取设备视频流或者音频流,返回Promise对象
```javascript
navigator.mediaDevices.getUserMedia({
  audio: true,
  video: false
})
.then(function(stream) {
  /* 使用这个stream stream */
})
.catch(function(err) {
  /* 处理error */
})
```

### [AudioContext](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioContext)
AudioContext接口表示由链接在一起的音频模块构建的音频处理图，每个模块由一个AudioNode表示。音频上下文控制它包含的节点的创建和音频处理或解码的执行。
```javascript
const audioCtx = new AudioContext()
```

#### [AudioContext.createMediaStreamSource](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioContext/createMediaStreamSource)
一般从 navigator.getUserMedia 获得MediaStream对象实例, 然后来自MediaStream的音频就可以被播放和操作。
```javascript
navigator.mediaDevices.getUserMedia(options).then(stream => {
  const mediaStreamSource = audioContext.createMediaStreamSource(stream)
})
```

#### [AudioContext.createScriptProcessor](https://developer.mozilla.org/zh-CN/docs/Web/API/BaseAudioContext/createScriptProcessor)
用于操作音频数据
```javascript
// 创建一个音频分析对象，采样的缓冲区大小为0（自动适配），输入和输出都是单声道
const scriptProcessor = audioContext.createScriptProcessor(0, 1, 1)
scriptProcessor.onaudioprocess = audioProcessingEvent => {
  // audioProcessingEvent是一个对象，可以提供许多操作方法
  audioProcessingEvent.inputBuffer // 音频二进制数据
}
```

### [connect](https://developer.mozilla.org/zh-CN/docs/Web/API/AudioNode/connect)
将一个节点的输出连接到一个指定目标
```javascript
// 将音源输入与API处理连接起来
mediaStreamSource.connect(scriptProcessor)
scriptProcessor.connect(audioContext.destination)
```

### 总结
`MediaDevices`用于获取音频输入源，`AudioContext`用于创建音频处理对象，两者使用`connect`连接起来。
