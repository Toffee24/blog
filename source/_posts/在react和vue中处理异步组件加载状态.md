---
title: 在vue和react中处理异步组件加载状态
date: 2019-09-24 14:49:23
tags:
---

### 异步组件加载与webpack SplitChunks
在我们平时的开发工作中，为了减少首屏代码体积，往往会把一些非首屏的组件设计成异步组件，按需加载。利用webpack对代码进行分割是懒加载的前提，懒加载就是异步调用组件，需要时候才下载（按需加载）。
通过异步组件结合webpack的代码按需加载，我们可以得到两点好处：
- 用不到的组件不会加载，因此网页打开速度会很快，当你用到这个组件的时候，才会通过异步请求进行加载；
- 缓存组件，通过异步加载的组件会缓存起来，当你下一次再用到这个组件时，丝毫不会有任何的疑迟，组件很快会从缓存中加载出来。

关于webpack splitChunks,可以阅读腾讯IMWeb的文章,介绍的非常详细了,[点击链接](https://imweb.io/topic/5b66dd601402769b60847149)
异步加载虽然由以上的好处,但是必然有一个问题,就是在浏览器下载异步组件的代码时(比如新页面路由切换),如果网速比较慢的话,会显示白屏界面。那么,如何实现在浏览器下载异步组件的同时让页面展示诸如loading类的等待界面呢?

### vue中处理异步组件加载状态
#### 声明方法
```javascript
/**
 * 处理路由页面切换时，异步组件加载过渡的处理函数
 * @param  {Object} AsyncView 需要加载的组件，如 import('@/components/home/Home.vue')
 * @return {Object} 返回一个promise对象
 */
function lazyLoadView (AsyncView) {
  const AsyncHandler = () => ({
    // 需要加载的组件 (应该是一个 `Promise` 对象)
    component: AsyncView,
    // 异步组件加载时使用的组件
    loading: import('@/components/public/RouteLoading.vue'),
    // 加载失败时使用的组件
    error: import('@/components/public/RouteError.vue'),
    // 展示加载时组件的延时时间。默认值是 200 (毫秒)
    delay: 200,
    // 如果提供了超时时间且组件加载也超时了，
    // 则使用加载失败时使用的组件。默认值是：`Infinity`
    timeout: 10000
  });
  return Promise.resolve({
    functional: true,
    render (h, { data, children }) {
      return h(AsyncHandler, data, children);
    }
  });
}
```
<!-- more -->
#### 引入路由并使用
```javascript
const helloWorld = () => lazyLoadView(import('@/components/helloWorld'))

{
	routes: [
        {
            path: '/helloWorld',
            name: 'helloWorld',
            component: helloWorld
        }
    ]
}
```
[相关文档链接](https://cn.vuejs.org/v2/guide/components-dynamic-async.html#%E5%A4%84%E7%90%86%E5%8A%A0%E8%BD%BD%E7%8A%B6%E6%80%81)

### react中处理异步组件加载状态
#### react-loadable
react-loadable是基于react的高阶组件,它专门用来处理异步路由加载状态,并且可以用于SSR服务端渲染
[相关文档链接](https://github.com/jamiebuilds/react-loadable)
#### 使用react-loadable
```jsx harmony
import Loadable from 'react-loadable';
import Loading from './my-loading-component';

const LoadableComponent = Loadable({
  loader: () => import('./my-component'),
  loading: Loading,
  delay: 200,
  timeout: 10000,
});

export default class App extends React.Component {
  render() {
    return <LoadableComponent/>
  }
}
```
