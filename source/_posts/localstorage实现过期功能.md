---
title: hack localstorage实现过期功能
date: 2019-12-05 10:31:32
tags:
---
### Window.localStorage
> 只读的'localStorage'属性允许你访问一个'Document'源（origin）的对象'Storage'；存储的数据将保存在浏览器会话中。'localStorage' 类似 'sessionStorage'，但其区别在于：存储在 'localStorage' 的数据可以长期保留；而当页面会话结束——也就是说，当页面被关闭时，存储在 'sessionStorage' 的数据会被清除 。

### localStorage用法
```javascript
localStorage.setItem('myCat', 'Tom');
localStorage.getItem('myCat');
localStorage.removeItem('myCat');
// 移除所有
localStorage.clear();
```

### hack
```javascript
(function () {
  var getItem = localStorage.getItem.bind(localStorage)
  var setItem = localStorage.setItem.bind(localStorage)
  var removeItem = localStorage.removeItem.bind(localStorage)
  localStorage.setItem = function (keyName, keyValue, expires) {
     if (typeof expires !== 'undefined') {
        var expiresDate = new Date(expires).valueOf()
        setItem(keyName + '_expires', expiresDate)
     }
    return setItem(keyName, keyValue)
  }
  localStorage.getItem = function (keyName) {
    var expires = getItem(keyName + '_expires')
    if (expires && new Date() > new Date(Number(expires))) {
      removeItem(keyName)
      removeItem(keyName + '_expires')
    }
    return getItem(keyName)
  }
})() 
```
原理: 
先看```localStorage.setItem```方法,此方法接收第三个参数`expires`过期时间,该值为一个时间戳, 并且会同时在缓存中设置一个后缀为`_expires`的key,value为过期时间。
再看```localStorage.getItem```方法,先判断是否有取值的后缀名为`_expires`的值,如果有,再判断当前时间戳是否大于取值的时间戳,如果大于的话,就执行删除`removeItem
`方法,将缓存中的两个值都删除,如果没有取到或者时间未到过期时间,则正常返回值。
