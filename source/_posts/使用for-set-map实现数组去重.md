---
title: '使用for,set,map实现数组去重'
date: 2018-11-13 09:33:07
tags:
---

数组去重的场景在开发中还是经常遇见的,比如一个地图选房功能,在地图上标识一个区域,显示区域上的房型,移动区域后显示新增的房型以及原来还在区域内的房型保持不变,实现这一个场景,就需要得出每次不变的房型的数组,以及新增的房型的数组.一般的,后台返回的数据只是我们选中的区域的房型数据,我们要对数据进行加工处理.

### 构造数据
让我们先来生成模拟数据,并打乱我们生成的数据,模拟真实情况
```javascript
function generateHouses(n) {
    //生成数据
    let newHouse = new Array(n)
    let oldHouse = new Array(n)
    for (let i = 0; i < n / 2; i++) {
        newHouse[i] = oldHouse[i] = {
            id: i
        }
    }
    for (let i = n / 2; i < n; i++) {
        newHouse[i] = {
            id: n * 2 + i
        }
        oldHouse[i] = {
            id: n + i
        }
    }
    return {
        newHouse: randomArr(newHouse),
        oldHouse: randomArr(oldHouse)
    }
}

function randomArr(arr) {
    //数组洗牌
    for (var i = arr.length - 1; i >= 0; i--) {
        var randomIndex = Math.floor(Math.random() * (i + 1));
        var itemAtIndex = arr[randomIndex];
        arr[randomIndex] = arr[i];
        arr[i] = itemAtIndex;
    }
    return arr
}
const {newHouse,oldHouse} = generateHouses(n) //n为需要构造的数据量
const newSetHouse = new Set(newHouse)  //构造set数据
const newMapHouse = new Map() //构造map数据
    for(let index in newHouse){
        newMapHouse.set(newHouse[index].id,newHouse[index])
    }
```

### For循环
for循环方法进行两次循环,因为每判断一次新数据就需要遍历一遍老数据,所以时间复杂度为O(N^2),对于1000的数据,最坏的情况要判断1000000次
```javascript
function filterHouse_for(newHouse, oldHouse) {
    if (newHouse.length === 0) {
        return {
            remainHouse: oldHouse,
            newHouse: newHouse
        }
    }
    let remainHouse = []
    let newOutPutouse = []
    for (let nIndex in newHouse) {
        let isNew = true
        for (let oIndex in oldHouse) {
            if (newHouse[nIndex].id === oldHouse[oIndex].id) {
                remainHouse.push(newHouse[nIndex])
                isNew = false
                break
            }
        }
        if (isNew) {
            newOutPutouse.push(newHouse[nIndex])
        }
    }
    return {
        remainHouse,
        newOutPutouse
    }
}
```

### Set方法
set是ES6新增的一种数据类型,它的成员是唯一的,所以用来进行去重是非常合适的,总体时间复杂度相较于for循环,它变成了O(NlogN),所以与for循环的时间长短比较取决于系数N,当系数较小时,差别不是很大,但当系数变大时,for循环消耗的时间是成指数级别增加的
```javascript
function filterHouse_set(newHouse, oldHouse,newSetHouse) {
    let remainHouse = []
    let newOutPutouse = []
    for (let index in oldHouse) {
        if (newSetHouse.has(oldHouse[index])) {
            remainHouse.push(oldHouse[index])
        } else {
            newOutPutouse.push(oldHouse[index])
        }
    }
    return {
        remainHouse,
        newOutPutouse
    }
}
```

### Map方法
哈希的时间复杂度为O(1)，因此总的时间复杂度为O(N)，代价是哈希的存储空间通常为数据大小的两倍,并且需要手动先构造map数据
```javascript
function filterHouse_map(newHouse,oldHouse,newMapHouse){
    let remainHouse = []
    let newOutPutouse = []
    oldHouse.map(ele=>{
        newMapHouse.has(ele.id)?remainHouse.push(ele):newOutPutouse.push(ele)
    })
    return {
        remainHouse,
        newOutPutouse
    }
}
```

### 时间比较
写一个函数来比较三种方法需要的时间
```JavaScript
function test(n){
    const {
        newHouse,
        oldHouse
    } = generateHouses(n)
    /* for */
    console.log('for Start')
    var time = new Date().getTime()
    filterHouse_for(newHouse, oldHouse)
    console.log('for End', new Date().getTime() - time)

    /* set */
    const newSetHouse = new Set(newHouse)  //构造set数据
    console.log('set Start')
    var time = new Date().getTime()
    filterHouse_set(newHouse, oldHouse,newSetHouse)
    console.log('set End', new Date().getTime() - time)

    /* map */
    const newMapHouse = new Map() //构造map数据
    for(let index in newHouse){
        newMapHouse.set(newHouse[index].id,newHouse[index])
    }
    console.log('map Start')
    var time = new Date().getTime()
    filterHouse_map(newHouse, oldHouse,newMapHouse)
    console.log('map End', new Date().getTime() - time)
}
```

### 运行结果(chrome下运行(版本 70))
当 n 为10时:
```
test(100)
VM253:97 for Start
VM253:100 for End 3
VM253:104 set Start
VM253:107 set End 0
VM253:114 map Start
VM253:117 map End 0
```
当 n 为1000时:
```
for Start
VM253:100 for End 60
VM253:104 set Start
VM253:107 set End 1
VM253:114 map Start
VM253:117 map End 0
```
当 n 为10000时:
```
for Start
VM253:100 for End 4532
VM253:104 set Start
VM253:107 set End 5
VM253:114 map Start
VM253:117 map End 2
```
当 n 为100000时:
因为for循环等了几分钟都没有算出结果,就不比较for循环了
```
set Start
VM231:102 set End 37
VM231:109 map Start
VM231:112 map End 30
```
当 n 为10000000时:
```
test(1000000)

VM231:99 set Start
VM231:102 set End 466
VM231:109 map Start
VM231:112 map End 592
```

### 结论
可以看出,不论哪种情况,for循环用时都是最长的,所以不要使用for循环😋,数据量是十万级别以下时,使用map速度比set快一些,十万级别以上时使用set
