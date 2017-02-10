---
layout: post
title:  "掌握vue中的组件"
date:   2016-07-31
categories: vue
tags: vue
---

定义组件的主要目的就是为了代码复用。在实际中，组件之间存在嵌套关系，难点就是弄清楚他们之间是怎么通信的。
vue借鉴了react定义组件的方式，也提倡单项绑定。因为单项绑定的数据流很清晰，很容易维护。

<!--more-->

下面例子parent中使用了自定义组件my-component:

1. `:message`传递的是动态属性，可以理解为引用传递。它与angular中的`{message:'='}`类似；
2. `color`属于属性值传递。它与angular中的`{message:'@'}`类似；
3. 子组件传值给父组件的做法：a.在子组件中派发事件（dispatch），该事件沿着父链冒泡； b.父组件监听事件。该例子子组件派发一个叫'child-event'的事件，父组件通过自定义事件的方式来监听`@child-event="handleIt"`，父组件收到child-event事件后就会调用methods中的handleIt方法。这是推荐的写法，因为它能清楚的知道child-event来自my-component。
4. slot分发内容。在定义组件的时候可以预留出一些卡槽，真正在使用的时候可以向里面插入内容。`slot="a"`中的内容就会插入到`slot name="a"`中。

```html
<template id="sub">
  <div>
    type is {{type}},message is {{message}}, color is {{color}}
    <button @click="notify">Dispatch Event</button>
    <slot></slot>
    <slot name="b"></slot>
    <slot name="a"></slot>
  </div>
</template>

<div id="parent">
  <div><input type="text" v-model="data"></div>
  This is parent value: {{message}}, recevie event value is {{dispatchValue}}
  <my-component :message="data" color="red" @child-event="handleIt">
    <div>This is some original content;</div>
    <div slot="a">This is content from a;</div>
    <div slot="b">This is content from b;</div>
  </my-component>
</div>
```

```javascript
Vue.component('my-component', {
  props: ['message', 'color'],
  template: '#sub',
  // 通常返回一个函数，避免所有实例共享同一对象
  data: function(){
    return {
      type: 'sub'
    }
  },
  methods: {
    notify: function(){
      // 派发事件
      this.$dispatch('child-event', "dispatch-value");
    }
  }
});

new Vue({
  el: '#parent',
  data: {
    data: '',
    dispatchValue: '',
    message: 'hello'
  },
  methods: {
    handleIt: function(msg){
      // 处理事件传递的msg
      this.dispatchValue = msg;
    }
  }
});
```

下面例子讲解了动态组件的使用方式:

1. 在components中定义需要动态切换的组件。
2. keep-alive会在内存中记住每个组件的状态，可以避免重新渲染。
3. is它其实就是条件控制
4. activate钩子，组件切入前，这里可以写一些业务逻辑。
5. transition-mode 过渡的动画方式。

```html
<div id="dynamic">
  <div><a @click="current='home'">Home</a> | <a @click="current='about'">About</a></div>
  <component 
    :is="current" 
    keep-alive
    transition="fade"
    transition-mode="out-in"
  ></component>
</div>
```

```javascript
new Vue({
  el: '#dynamic',
  data: {
    current: 'home'
  },
  components: {
    home: {
      template: '<span @click="add">home:{{num}}</span>',
      data: function(){
        return {num: 0}
      },
      // 组件切入前的钩子函数
      activate: function(done){
        setTimeout(()=>{this.num = 5; done()},1000);
      },
      methods: {
        add: function(){
          this.num++;
        }
      }
    },
    about: {
      template: '<div>about</div>'
    }
  }
});
```



