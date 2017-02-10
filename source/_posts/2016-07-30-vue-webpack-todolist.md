---
layout: post
title:  "使用vue、webpack实现简单todolist"
date:   2016-07-30 
categories: vue
tags: [vue, webpack]
---

如果你熟悉Angular，会发现上手vue很容易。确实这两者中的一些概念很类似，如双向绑定、过滤器、事件、条件控制、表达式等。

<!--more-->

一般学习一个类库或者框架，最好是从简单的todolist入手，下面直接上代码

先看一下webpack的配置

```javascript
module.exports = {
  // 入口
  entry: './app.js',
  output: {
    // 处理后的输出
    filename: 'bundle.js'
  },
  module: {
    loaders: [{
      test: /\.js[x]?$/, 
      exclude: /node_modules/, 
      // 使用babel将ES6编译为ES5
      loader: 'babel-loader?presets[]=es2015'
    },{
      // 处理css
      test: /\.css$/, 
      loader: 'style-loader!css-loader'
    }]
  }
};
```

下面是html

```html
<section id="todoapp">
  <header id="header">
      <h1>todos</h1>
      <form id="todo-form" v-on:submit.prevent="save">
          <input id="new-todo" placeholder="What needs to be done?" autofocus="" v-model="todoInput">
      </form>
  </header>
  <section id="main">
      <ul id="todo-list">
        <li v-for="todo in todos | filterBy filterTodos" :class="{completed: todo.completed}">
          <div class="view">
            <input class="toggle" type="checkbox" v-model="todo.completed">
            <label>{{todo.name}}</label>
            <button class="destroy" @click="deleteTodo(todo)"></button>
          </div>
        </li>
      </ul>
  </section>
  <footer id="footer" v-show="unCompletedLength">
    <span id="todo-count">
      <strong>{{unCompletedLength}}</strong>items left
    </span>
    <ul id="filters">
        <li v-for="item in filters">
            <a @click="filterBy=item" :class="{'selected': filterBy==item}">{{item}}</a>
        </li>
    </ul>
    <button id="clear-completed" @click="clear">Clear completed</button>
  </footer>
</section>
```

下面是程序主要逻辑，比较简单

```javascript
import Vue from 'vue';
import Todo from './todo';

require('../css/base.css');
require('../css/index.css');

export default new Vue({
  el: '#todoapp',
  data: {
    filters: ['All', 'Active', 'Completed'], //过滤条件
    filterBy: 'All', //过滤的属性
    todoInput: '', //新创建todo的model
    todos: [] //列表
  },
  computed: {
    unCompletedLength: function(){
      return this.todos.filter((o) => {return !o.completed;}).length;
    }
  },
  methods: {
    save: function(){
      if(this.todoInput){
        this.todos.push(new Todo(this.todoInput));
        this.todoInput = "";
      }
    },
    deleteTodo: function(todo){
      this.todos.$remove(todo);
    },
    filterTodos: function(o){
      return this.filterBy == 'Active' ? !o.completed :
        this.filterBy == 'Completed' ? o.completed :
        o;
    },
    clear: function(){
      this.todos = this.todos.filter((o) => {return !o.completed;});
    }
  }
});
```