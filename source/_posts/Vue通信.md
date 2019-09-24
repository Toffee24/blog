---
title: Vue通信
date: 2019-06-12 13:59:58
tags:
---

### props和$emit
`props`父传子，`$emit`子传父
```javascript
	Vue.component('child',{
        data(){
            return {
                mymessage:this.message
            }
        },
        template:`
            <div>
                <input type="text" v-model="mymessage" 
                @input="passData(mymessage)"> </div>
        `,
        props:['message'],//得到父组件传递过来的数据
        methods:{
            passData(val){
                //触发父组件中的事件
                this.$emit('getChildData',val)
            }
        }
    })
//┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┅┄＊
	Vue.component('parent',{
        template:`
            <div>
                <p>this is parent compoent!</p>
                <child :message="message"
                v-on:getChildData="getChildData"></child>
            </div>
        `,
        data(){
            return {
                message:'张不怂'
            }
        },
        methods:{
            //执行子组件触发的事件
            getChildData(val){
                console.log(val)
            }
        }
    })
//┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┅┄＊

	 // 挂载
    var app=new Vue({
        el:'#app',
        template:`
            <div>
                <parent></parent>
            </div>
        `
    })
```

### 中央事件总线new Bus
新建一个Vue事件bus对象，然后通过`bus.$emit`触发事件，`bus.$on`监听触发的事件
```javascript
Vue.component('brother1',{
        data(){
            return {
                mymessage:'hello brother1'
            }
        },
        template:`
            <div>
                <p>this is brother1 compoent!</p>
                <input type="text" v-model="mymessage"
                @input="passData(mymessage)"> 

            </div>
        `,
        methods:{
            passData(val){
                //触发全局事件globalEvent
                bus.$emit('globalEvent',val)

            }
        }
    })
 //┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┅┄＊

    Vue.component('brother2',{
        template:`
            <div>
                <p>this is brother2 compoent!</p>
                <p>brother1传递过来的数据：{{brothermessage}}</p>
            </div>
        `,
        data(){
            return {
                mymessage:'hello brother2',

                brothermessage:''
            }
        },
        mounted(){
            //绑定全局事件globalEvent
            bus.$on('globalEvent',(val)=>{
                this.brothermessage=val;
            })
        }
    })
//┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┅┄＊

    //中央事件总线
    var bus=new Vue();

    var app=new Vue({
        el:'#app',
        template:`
            <div>
                <brother1></brother1>
                <brother2></brother2>
            </div>
        `
    })
```
<!-- more -->
### provide和inject
父组件中通过provider来提供变量，然后在子组件中通过inject来注入变量。不论子组件有多深，只要调用了inject那么就可以注入provider中的数据。而不是局限于只能从当前父组件的prop属性来获取数据，只要在父组件的生命周期内，子组件都可以调用。
```javascript
Vue.component('child',{
        inject:['parent_var'],//得到父组件传递过来的数据
        data(){
            return {
                mymessage:this.parent_var
            }
        },
        template:`
            <div>
               {{ message }}
            </div>
    })
//┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┄┅┄┅┄┅┄┅┄＊
    Vue.component('parent',{
        template:`
            <div>
                <child></child>
            </div>
        `,
        provide:{
            /*
            比如你可以把用户登录信息存储在App.vue中，可以把
            provide:{app:this}注入，后续所有组件通过inject:['app'],
            就可以直接通过app.userInfo拿到用户信息
            */
            parent_var:'随便什么都可以（可以是this,可以是data中的数据）'
        },
        data(){
            return {
                message:'hello'
            }
        }
    })
```

### $attrs和$listeners
还是多层的场景，App.vue-->A—>B,如果App直接想给B传递数据该怎么办？`Vue 2.4`开始提供了`$attrs`和`$listeners`来解决这个问题，能够让组件App直接传递数据给组件B
app.vue引入A组件
```html
<template>
  <div id="app">
    {{app}}
    // ******关键点*****
    <A :app="app" @test="doTest"/>
  </div>
</template>

<script>
import A from "./components/A";

export default {
  name: "App",
  data() {
    return {
      app: "我是App的数据"
    };
  },
  components: {
    A
  },
  methods: {
    doTest() {
      console.log(this.app)
    }
  }
}
```
A.vue引入B组件
```html
<template>
  <div class="hello">
    <h6>这里是A组件</h6>
    <p>$attrs: {{$attrs}}</p>
    <p>$listeners: {{$listeners}}</p>
     // ******关键点***** v-bind传递的都是$attrs,v-on传递的都是$listeners
    <B v-bind="$attrs" v-on="$listeners"/>
  </div>
</template>

<script>
import B from "./B";

export default {
  name: "A",
  props: {
    msg: String
  },
  components: { B },
  mounted() {
    console.log(this.$listeners);
  }
};
</script>
```
B组件
```html
<template>
  <div class="hello">
    <h6>这里是B组件</h6>
     // ******关键点*****
    <p>$attrs: {{$attrs}}</p>
  </div>
</template>

<script>

export default {
  name: "B",
  props: {
    msg: String
  },
  mounted() {
   // ******关键点*****
   // 为啥这里直接能emitApp组件传递的test呢？
   // 因为在A组件中有一个关键操作是  <B v-bind="$attrs" v-on="$listeners"/>
    this.$emit("test");
  }
};
</script>
```

### $parent和$children
分别是获得父组件和子组件的实例
```javascript
Vue.component('child',{
        props:{
            value:String, //v-model会自动传递一个字段为value的prop属性
        },
        data(){
            return {
                mymessage:this.value
            }
        },
        methods:{
            changeValue(){
                this.$parent.message = this.mymessage;//通过如此调用可以改变父组件的值
            }
        },
        template:`
            <div>
                <input type="text" v-model="mymessage" @change="changeValue"> 
            </div>
    })
    Vue.component('parent',{
        template:`
            <div>
                <p>this is parent compoent!</p>
                <button @click="changeChildValue">test</button >
                <child></child>
            </div>
        `,
        methods:{
            changeChildValue(){
                this.$children[0].mymessage = 'hello';
            }
        },
        data(){
            return {
                message:'hello'
            }
        }
    })
    var app=new Vue({
        el:'#app',
        template:`
            <div>
                <parent></parent>
            </div>
        `
    })
```

### v-model
父组件通过v-model传递值给子组件时，会自动传递一个value的prop属性，在子组件中通过this.$emit(‘input’,val)自动修改v-model绑定的值
```javascript
Vue.component('child',{
        props:{
            value:String, //v-model会自动传递一个字段为value的prop属性
        },
        data(){
            return {
                mymessage:this.value
            }
        },
        methods:{
            changeValue(){
                this.$emit('input',this.mymessage); //通过如此调用可以改变父组件上v-model绑定的值
            }
        },
        template:`
            <div>
                <input type="text" v-model="mymessage" @change="changeValue"> 
            </div>
    })
    Vue.component('parent',{
        template:`
            <div>
                <p>this is parent compoent!</p>
                <p>{{message}}</p>
                <child v-model="message"></child>
            </div>
        `,
        data(){
            return {
                message:'hello'
            }
        }
    })
    var app=new Vue({
        el:'#app',
        template:`
            <div>
                <parent></parent>
            </div>
        `
    })
```

### Vue.observable()
`vue2.6`之前，创建一个响应对象，必须在一个Vue实例中配置。现在我们可以在Vue实例外部，通过使用Vue.observable(data)创建，如下
```javascript
import vue from vue;
const state = Vue.observable ({
   counter: 0,
});
export default {
   render () {
     return (
       <div>
         {state.counter}
           <button v-on:click={() => {state.counter ++; }}>
           Increment counter
         </ button>
       </ div>
     );
   },
};
```

### vuex
具体参考[文档](https://vuex.vuejs.org/zh/guide/)
