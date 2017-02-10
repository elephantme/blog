---
layout: post
title:  "使用vue+vue-route+webpack构建单页面应用"
date:   2016-08-01
categories: vue
tags: [vue, webpack]
---

vue-route是vue官方的一个插件，看了文档之后，觉得他设计得相当棒，提供了一些钩子方法，可以方便的实现各种业务。我会单独写一篇文章来谈谈。

<!--more-->

该例子主要功能是

1. 默认进来是产品列表
2. 单击产品列表后进入功能面板，包含玩家和任务两个功能菜单
3. 功能面板header提供搜索功能，该搜索条件将会全局生效

下面是项目结构

```
├── components
│   ├── AppList.vue
│   ├── MainSkeleton.vue
│   ├── Player.vue
│   └── Task.vue
├── services
│   └── AppService.js
├── app.js
├── bundle.js
├── index.html
└── webpack.config.js
```

1. components中存放的都是vue组件，也就是每个页面。 其中MainSkeleton是功能面板的骨架，Player和Task会嵌套在其中。
2. services中存放的是具体的业务逻辑，例如从服务器端请求数据等等。 
3. app.js是程序的入口。
4. index.html就是单页入口。
5. bundle.js是webpack打包后的文件。

webpack.config.js

```javascript
module.exports = {
  // 入口
  entry: './app.js',
  output: {
    // 处理后的输出
    filename: 'bundle.js'
  },
  module: {
    loaders: [
      {
        test: /\.js[x]?$/, 
        exclude: /node_modules/, 
        // 使用babel将ES6编译为ES5
        loader: 'babel-loader?presets[]=es2015'
      },
      {
        test: /\.vue$/,
        loader: 'vue'
      }
    ]
  }
};
```

可以看到使用了babel和vue的loader，用来将ES6便以为ES5及处理.vue文件。

app.js

```javascript
import Vue from 'vue';
import VueRouter from 'vue-router';
import AppList from './components/AppList.vue';
import MainSkeleton from './components/MainSkeleton.vue';
import Player from './components/Player.vue';
import Task from './components/Task.vue';

//使用vue-router插件
Vue.use(VueRouter);

var App = Vue.extend({});

var router = new VueRouter();

router.map({
  '/': {
    component: AppList
  },
  '/app': {
    component: MainSkeleton,
    subRoutes: {
      '/player': {
        component: Player
      },
      '/task': {
        component: Task
      }
    }
  }
});

router.start(App, '#app');
```

主要是定义了router的配置（项目负责的话可以单独写在router.js中），定义每个路由对应的组件。Angular中的router会将路由映射到template和controller中。在vue中组件其实就包括了template和controller的逻辑。

AppList.vue

```html
<template>
  <h1>App List</h1>

  <div>
    <ul>
      <li v-for="item in apps" @click="viewDetail(item.id)">{{item.name}}</li>
    </ul>
  </div>
</template>

<script>
import AppService from '../services/AppService';

export default {
  data: function(){
    return {
      apps: []
    }
  },
  route:{
    data: function(transition){
      transition.next({
        apps: AppService.getAllApps()
      });
    }
  },
  methods: {
    viewDetail: function(id){
      this.$route.router.go('/app/player')
    }
  }
};
</script>
```

没错模板和业务逻辑甚至样式都在一个文件中，vue提倡这种做法，理由是他们是一个整体。我觉得这样也挺好，不过我可能会将css移动到单独的css文件中。

route中提供了好多钩子函数，会在当前组件载出、载入时调用。其中常用的就是data，只要hash值发生变化，他就会被调用。所以他常用来加载和设置当前组件的数据。

MainSkeleton.vue

```html
<template>
  <header>
    <a v-link="{path: '/'}">返回</a>
    从<input type="date" v-model="start">至<input type="date" v-model="end">
    <input type="button" @click="search" value="查询">
  </header>
  <section class="menu">
    <ul>
      <li @click="switchMenu('/app/player')">玩家</li>
      <li @click="switchMenu('/app/task')">任务</li>
    </ul>
  </section>
  <section class="main">
    <router-view></router-view>
  </section>
</template>

<script>
  export default {
    data: function(){
      return {
        start: '',
        end: ''
      }
    },
    methods: {
      // 全局搜索条件
      search: function(){
        this.go(this.$route.path);
      },
      // 左侧导航切换
      switchMenu: function(path){
        this.go(path);
      },
      go: function(path){
        this.$route.router.go({
          path: path, 
          query: {
            start: this.start,
            end: this.end
          }
        });
      }
    }
  }
</script>

<style>
  header{
    padding:10px;
  }
  .menu{
    width: 200px;
    float: left;
  }
  .main{
    margin-top:20px;
    float: left;
  }
</style>
```

该组件是功能面板的骨架，中间功能需要嵌套其它组件。其中header部分是搜索条件，左侧是菜单导航，右侧是功能区。在vue-router没发现有router.reload的api，就使用了`this.$route.router.go()`，path还是当前的path，只是将query设置为要查询的条件，在switch menu的时候就会将该搜索条件带上，这样每个页面都会应用到。

Player.vue

```html
<template>
  This is player page;
</template>

<script>
export default {
  route: {
    data: function(){
      console.log('execute player logic.., query params is ' + JSON.stringify(this.$route.query))
    }
  }
}
</script>
```
route.data 钩子里存放获取组件依赖的数据逻辑。 

可以看到其实是编写各种组件，大到每个页面，小到页面中的下来列表等等。本文没有涉及到ajax请求，有相关的第三方插件vue-resources。

<a href="https://github.com/elephantme/learn_todolist/tree/master/vue-router" target="_blank">可以前往查看项目地址</a>






