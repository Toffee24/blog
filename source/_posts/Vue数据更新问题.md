---
title: Vue数据更新问题
date: 2019-06-11 20:05:08
tags:
---

### Vue 的数组更新问题

> 以下参考 Vue 文档
  由于 JavaScript 的限制，Vue 不能检测以下数组的变动：
  1.当你利用索引直接设置一个数组项时，例如：vm.items[indexOfItem] = newValue
  2.当你修改数组的长度时，例如：vm.items.length = newLength

**题外话** 实际上，我们在 `Vue` 的数组书使用 `splice`、`push`等方法，`Vue` 都已经做了一层封装，所以它们才能出发视图更新，如果有想更加深入了解，可以阅读[源码](https://ustbhuangyi.github.io/vue-analysis/prepare/flow.html#%E4%B8%BA%E4%BB%80%E4%B9%88%E7%94%A8-flow)。

### Vue 强制刷新——$forceUpdate()
> 如果你发现你自己需要在 Vue 中做一次强制更新，99.9% 的情况，是你在某个地方做错了事。

类似的代码如下：
```javascript
// 在控制变量改变的时候进行 强制渲染更新

let childrenRefs = this.$refs.elTabs.$children
this.$nextTick(() => {
  childrenRefs.forEach(child => child.$forceUpdate())
})

```


### 深拷贝数据
先用一个数据深拷贝数据，这里使用了 `slice` 方法，然后置空，最后在 `$nextTick` 中赋值深拷贝出来的数组值。
```javascript
  var newArray = this.questionData.slice(0)
  this.questionData = []
  this.$nextTick(function () {
    this.questionData = newArray;
  })
```
