---
title: RxJs实现基本拖拽
date: 2019-01-04 14:53:35
tags:
---
### HTML
```html
<div id="anchor">
    <div class="video" id="video">
        <video width="100%" height="100%" controls >
            <source src="http://download.blender.org/peach/bigbuckbunny_movies/big_buck_bunny_480p_stereo.ogg"
                    type="video/ogg">
            Your browser does not support HTML5 video.
        </video>
    </div>
</div>
```

### CSS
```css
<style>
        * {
            -webkit-box-sizing: border-box;
            -moz-box-sizing: border-box;
            box-sizing: border-box; }

        html, body {
            margin: 0;
            padding: 0;
            height: 2000px;
            background-color: #eee; }

        #anchor {
            height: 360px;
            width: 100%;
            background-color: #F0F0F0; }

        .video {
            width: 640px;
            height: 360px;
            margin: 0 auto;
            background-color: black; }
        .video.video-fixed {
            position: fixed;
            top: 10px;
            left: 10px;
            width: 320px;
            height: 150px;
            cursor: all-scroll; }
        .video.video-fixed .masker {
            display: none; }
        .video.video-fixed:hover .masker {
            display: block;
            position: absolute;
            width: 100%;
            height: 180px;
            background-color: rgba(0, 0, 0, 0.8);
            z-index: 2; }

    </style>
```

### JS
```javascript
    import {Observable} from 'rxjs/Rx'

    const $video = document.querySelector('#video')
	const $anchor = document.querySelector('#anchor')
	const scroll = Observable.fromEvent(document, 'scroll').    //监听document事件
		map(e => $anchor.getBoundingClientRect().bottom < 0).  //map将event事件映射为判断式布尔值
		subscribe(bool => {                                   //如果当前元素不在视窗内,则改变样式
			if(bool){
				$video.classList.add('video-fixed')
			}else{
				$video.classList.remove('video-fixed')
				$video.style.transform = ''
			}
		})

	const mouseDown = Observable.fromEvent($video, 'mousedown') //observe事件
	const mouseMove = Observable.fromEvent(document, 'mousemove')
	const mouseUp = Observable.fromEvent(document, 'mouseup')

	const videoBoundingRect = $video.getBoundingClientRect() //获取viedo元素rect对象
	const validValue = (value,max,min)=>{                    //比较函数,防止拖拽出屏幕
		return Math.min(Math.max(value,min),max)
	}
	mouseDown.filter(e => $video.classList.contains('video-fixed')). //filter过滤
		switchMap(e => mouseMove.takeUntil(mouseUp)). //将高阶Oberve转换为低阶
		withLatestFrom(mouseDown, (move, down) => {  //结合上一个observable源的结果
			return {  //move.clientX - down.offsetX =>处理抖动
				x: validValue(move.clientX - down.offsetX,window.innerWidth - videoBoundingRect.width/2,0),
				y: validValue(move.clientY - down.offsetY,window.innerHeight - videoBoundingRect.height/2,0),
			}
		}).
		subscribe(e => {
			$video.style.transform = `translate3D(${e.x}PX,${e.y}px,0)`
		})

```
