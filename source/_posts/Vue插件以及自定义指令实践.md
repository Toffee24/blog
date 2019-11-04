---
title: Vue插件以及自定义指令实践
date: 2019-11-04 16:46:06
tags:
---

### 背景
由于项目需要引入`countup.js`插件用于将数字实现累加动画效果,而且由于是vue项目,发现文档里有专门对vue封装的组件`vue-countup-v2`,所以自然而然
`yarn add countup.js vue-countup-v2`
然后根据文档一步一步下来,一切ok
随后想了想,需要安装两个依赖并且` vue-countup-v2`只是一个组件封装,那么为什么不用`vue directives`指令来写呢?

### vue-directives自定义指令
vue指令其实随处都会用到,比如`v-if`,`v-show`,`v-for`等等,对于特殊场景,vue也提供注册自定义指令的方法,有两种方法可以注册自定义指令,分别是`Vue.directive`注册全局指令以及在组件中配置`directives`注册局部指令,在这里,全局指令更适合本场景,所以决定使用`Vue.directive`

### 开始编写
```javaScript
// main.js
import { CountUp } from 'countup.js'
Vue.directive('countUp', {
            bind (el) {
                el.countUpComponent = new CountUp(el, 0, {
                    duration: 1
                })
            },
            update (el, binding) {
                el.countUpComponent.update(binding.value || 0)
            }
        })
```
<!-- more -->
在这里,`bind`方法只调用一次，指令第一次绑定到元素时调用。在这里可以进行初始化`Countup`对象,之所以不用`inserted`方法是因为根据文档描述,`inserted`方法并不一定可以获取到dom对象示例,这就会导致`new Countup`示例时报错
在`el`对象上挂载实例对象,这样,在`update`触发时,可以执行`update`方法触发数字更新动画
这里,`binding`对象可以获取到我们传给指令的参数,进而执行`update`方法

### vue插件编写
其实,上述代码已经可以实现功能了,但是为了降低代码的耦合性,我们可以将它封装成一个插件
写vue插件需要暴露一个`install`方法,这个方法接收两个参数,第一个参数是`Vue`构造器，第二个参数是一个可选的选项对象
```JavaScript
// countUp-directive.js
import { CountUp } from 'countup.js'

export default {
    install (Vue, options) {
        Vue.directive('countUp', {
            bind (el) {
                el.countUpComponent = new CountUp(el, 0, {
                    duration: 1
                })
            },
            update (el, binding) {
                el.countUpComponent.update(binding.value || 0)
            }
        })
    }
}
```
```javaScript
// main.js
import CountUp from './countup-directive'
Vue.use(CountUp)
```

### 使用
使用就很简单了,直接看代码
```
<em v-countUp="countNumber"></em>
```
传入`countNumber`就可以了

### 相关文档
[Vue自定义指令](https://cn.vuejs.org/v2/guide/custom-directive.html)
[Vue插件](https://cn.vuejs.org/v2/guide/plugins.html) 
[countup.js](https://github.com/inorganik/countUp.js/)