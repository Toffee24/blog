---
title: 【翻译】如何用puppeteer通过slider滑块认证
date: 2019-08-22 15:25:32
---

![image](/blog/images/1_8d-YL34nSPjKw0RQk8xuvg.jpeg)
<figcaption style="font-size:12px;text-align:center;margin-top:-20px">Photo by James Pond on Unsplash</figcaption>

垃圾邮件对于网站所有者来说是个大问题。另一方面，划块验证让我发疯，他们对用户体验不太友好。

划块验证虽然很糟糕但是我们还是要面对它,有很多种方法可以人机验证,但是他们一般来说都不太好。

近年来爬虫变得越来越智能并且网站很难去识别,在拥有大量的时间以及足够的资源后,我们几乎可以通过任何人机验证。现在有插件可以防止使用puppeteer被检测并解决人机验证,甚至出现了专门的公司。我们无法检测到被puppeteer控制的无头chrome浏览器。

有些网站使用了滑块验证来作为人机验证的一种方案,但是为什么有人会使用这么简单的东西来绕过它:
- 大多数爬虫不执行js代码
- 用户体验更好
- 滑块的方式天然适合手机用户
  
所以,滑块验证对于用户来说是美好且亲切的,但是如果只有滑块验证的话那就象手拿木棒赤裸地进入战场了。

让我们来通过滑块验证吧!

### 滑动提交
有一种jquery插件是专门为这种防止垃圾邮件的表单服务的
![image](/blog/images/1_0LXnPGyW3gHt_tKZlGL4Bg.png)

首先我们计算出滑块区域的边界信息,为了移动滑块,我们要做以下这些事:
  - 把鼠标放在要操作元素的中心
  - 按下鼠标
  - 移动
  - 在合适的位置松开鼠标

```javascript
const puppeteer = require('puppeteer')

async function run() {
    const browser = await puppeteer.launch({
        headless: false,
        defaultViewport: { width: 1366, height: 768 }
    })
    const page = await browser.newPage()
    await page.goto('http://kthornbloom.com/slidetosubmit/')
    await page.type('input[name="name"]', 'Puppeteer Bot')
    await page.type('input[name="email"]', 'js@automation.com')

    let sliderElement = await page.$('.slide-submit')
    let slider = await sliderElement.boundingBox()

    let sliderHandle = await page.$('.slide-submit-thumb')
    let handle = await sliderHandle.boundingBox()

    await page.mouse.move(handle.x + handle.width / 2, handle.y + handle.height / 2)
    await page.mouse.down()
    await page.mouse.move(handle.x + slider.width, handle.y + handle.height / 2, { steps: 10 })
    await page.mouse.up()

    await page.waitFor(3000)

    // success!

    await browser.close()
}

run()
```
<!-- more -->

### Dipbit注册滑块验证
Dipbit是一个数字货币兑换网站。登录页面和注册页面都有滑块验证。
![image](/blog/images/1_V72pGVtq63udlTwpzYPHuA.png)
Dipbit就要显得聪明一点了,所以我们要增加一些代码来隐藏puppeteer的执行
```JavaScript
const puppeteer = require('puppeteer')

async function run() {
    const browser = await puppeteer.launch({
        headless: false,
        defaultViewport: { width: 1366, height: 768 }
    })
    const page = await browser.newPage()

    await page.evaluateOnNewDocument(() => {
        Object.defineProperty(navigator, 'webdriver', {
            get: () => false //注 检测是否为无头浏览器
        })
    })

    await page.goto('https://www.dipbit.com/auth/login')
    await page.type('#email', 'js@automation.com')
    await page.type('#password', 'password123')

    let sliderElement = await page.$('.slidetounlock')
    let slider = await sliderElement.boundingBox()

    let sliderHandle = await page.$('.nc_iconfont.btn_slide')
    let handle = await sliderHandle.boundingBox()

    await page.mouse.move(handle.x + handle.width / 2, handle.y + handle.height / 2)
    await page.mouse.down()
    await page.mouse.move(handle.x + slider.width, handle.y + handle.height / 2, { steps: 50 })
    await page.mouse.up()

    await page.waitFor(3000)

    // success!

    await browser.close()
}

run()

```
![image](/blog/images/1_JzqzvD-nuy--t7kvfGgu3g.gif)

### 淘宝
淘宝的注册验证和Dipbit相似,唯一的区别就是他的验证表单嵌套在一个iframe的框架中,但对于Puppeteer来说,这不是问题。
```JavaScript
const puppeteer = require('puppeteer')

async function run() {
    const browser = await puppeteer.launch({
        headless: false,
        defaultViewport: { width: 1366, height: 768 }
    })
    const page = await browser.newPage()

    await page.evaluateOnNewDocument(() => {
        Object.defineProperty(navigator, 'webdriver', {
            get: () => false
        })
    })

    await page.goto('https://world.taobao.com/markets/all/sea/register')

    let frame = page.frames()[1]
    await frame.waitForSelector('.nc_iconfont.btn_slide')

    const sliderElement = await frame.$('.slidetounlock')
    const slider = await sliderElement.boundingBox()

    const sliderHandle = await frame.$('.nc_iconfont.btn_slide')
    const handle = await sliderHandle.boundingBox()
    await page.mouse.move(handle.x + handle.width / 2, handle.y + handle.height / 2)
    await page.mouse.down()
    await page.mouse.move(handle.x + slider.width, handle.y + handle.height / 2, { steps: 50 })
    await page.mouse.up()

    await page.waitFor(3000)

    // success!

    await browser.close()
}

run() 
```

![image](/blog/images/1_zQ7Sa5tr295jOxFLWaW1ng.gif)

### 拼图验证
我遇到了'滑动验证'vue组件应该对人来说很容易但是对机器不容易
此验证方法获取图像，创建2个画布和一个滑块。它使用拼图渲染初始图像。用户将移动滑块和拼图块匹配。当两个部分匹配时，用户应释放滑块并验证结束。
这种验证随机化拼图位置来混淆机器人。
![image](/blog/images/1_GMcgXCSRkGW7GORpTh543Q.png)
在此我不想做任何诸如机器学习或者OCR等花哨的东西,所以我只是一点一点的移动并与最初的图片进行比较
```JavaScript
const puppeteer = require('puppeteer')
const Rembrandt = require('rembrandt')

async function run () {
    const browser = await puppeteer.launch({
        headless: false,
        defaultViewport: { width: 1366, height: 768 }
    })
    const page = await browser.newPage()

    let originalImage = ''

    await page.setRequestInterception(true)
    page.on('request', request => request.continue())
    page.on('response', async response => {
        if (response.request().resourceType() === 'image')
            originalImage = await response.buffer().catch(() => {})
    })

    await page.goto('https://monoplasty.github.io/vue-monoplasty-slide-verify/')

    const sliderElement = await page.$('.slide-verify-slider')
    const slider = await sliderElement.boundingBox()

    const sliderHandle = await page.$('.slide-verify-slider-mask-item')
    const handle = await sliderHandle.boundingBox()

    let currentPosition = 0
    let bestSlider = {
        position: 0,
        difference: 100
    }

    await page.mouse.move(handle.x + handle.width / 2, handle.y + handle.height / 2)
    await page.mouse.down()

    while (currentPosition < slider.width - handle.width / 2) {

        await page.mouse.move(
            handle.x + currentPosition,
            handle.y + handle.height / 2 + Math.random() * 10 - 5
        )

        let sliderContainer = await page.$('.slide-verify')
        let sliderImage = await sliderContainer.screenshot()

        const rembrandt = new Rembrandt({
            imageA: originalImage,
            imageB: sliderImage,
            thresholdType: Rembrandt.THRESHOLD_PERCENT
        })

        let result = await rembrandt.compare()
        let difference = result.percentageDifference * 100

        if (difference < bestSlider.difference) {
            bestSlider.difference = difference
            bestSlider.position = currentPosition
        }

        currentPosition += 10
    }

    await page.mouse.move(handle.x + bestSlider.position, handle.y + handle.height / 2, { steps: 10 })
    await page.mouse.up()

    await page.waitFor(3000)

    // success!

    await browser.close()
}

run()
```

以防万一你错过了一个很酷的部分。我将Y轴上的滑块移动随机化以模拟真实的用户鼠标移动😎
```JavaScript
await page.mouse.move(
            handle.x + currentPosition,
            handle.y + handle.height / 2 + Math.random() * 10 - 5
        )
```

以上所有代码都在[这个仓库](https://github.com/fvitas/slider-anti-captcha)

### 结论
如果网站拥有更好的体验并且可以轻松绕过人机验证，或者网站积极地保护自己免受机器人攻击但是用户体验不佳，这一直是一个两难选择。
网站和机器人之间的战争永远不会结束。
无论网站采用何种验证方法，有人想出如何绕过它只是时间问题。

[原文链接](https://medium.com/@filipvitas/how-to-bypass-slider-captcha-with-js-and-puppeteer-cd5e28105e3c)
