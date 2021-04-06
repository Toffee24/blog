---
title: Art Glitch文字抖动效果实现
date: 2020-05-07 17:58:21
tags:
---

### 预览效果
<h4 class="main-text skew blockquote-center">
    <span class="back-text glitch">WELCOME TO TOFFEE'S BLOG</span>
</h4>
<!-- more -->

### 代码
```html
<h4 class="main-text skew">
    <span class="back-text glitch">WELCOME TO TOFFEE'S BLOG</span>
</h4>
```

```css
.main-text.skew {
            font-size: 36px;
            font-weight: bold;
            animation: skew .95s infinite alternate;
            background-image: linear-gradient( 78deg,#38215d,#c8266a 25%,#e579ac 47%,#8b959c 64%,#051f33 );
            position: relative;
            -webkit-background-clip: text;
        }
        .glitch {
            position: absolute;
            left: 0;
            animation: glitch .8s infinite alternate;
        }
        @keyframes skew {
            0%, 100%, 20%, 28%, 70%, 78% {
                transform: skew(0);
            }
            24% {
                transform: skew(-7deg);
            }
            74% {
                transform: skew(10deg);
            }
        }
        @keyframes glitch {
            0% {
                transform: translate3d(0,0,0);
                opacity: 1;
            }
            20% {
                transform: translate3d(0,0,0);
                opacity: .3;
            }
            24% {
                transform: translate3d(5px,4px,0);
                opacity: 1;
            }
            100%, 28%, 60%, 68% {
                transform: translate3d(0,0,0);
            }
            64% {
                transform: translate3d(-4px,-3px,0);
            }
            70%, 78% {
                opacity: 1;
                transform: translate3d(0,0,0);
            }
            74% {
                opacity: .3;
                transform: translate3d(10px,-6px,0);
            }
        }
```
