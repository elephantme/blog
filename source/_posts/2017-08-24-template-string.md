---
title: ES6中模板字符串详解
date: 2017-08-24 22:25:31
tags:
categories: ES6
---

> 模板字符串基础用法很简单，常用于变量内插。但它能和类似`Handlebars`这样的模板引擎相媲美...一起来看一下吧~

## 基础语法

具体可以参考之前写的一篇文章[《深入理解ES6-块级作用域和字符串》](https://elephantme.github.io/2017/07/11/es6-part-one/#模板字面量)

## 使用模板字符串实现模板引擎功能

先使用Handlebars来实现一个常见例子，如下：

```javascript
var Handlebars = require('handlebars');

// 工具方法
Handlebars.registerHelper('capitalize', function (str) {
  return str[0].toUpperCase() + str.slice(1);
});

// 编译模板
var page = Handlebars.compile(`
  <h2>People</h2>
  <ul>
  {{#each people}}
    <li>
      <span>{{capitalize name}}</span>
      {{#if ../isAdmin}}
        <button>Delete </button>
      {{/if}}
    </li>
  {{/each}}
  </ul>
`);

// 渲染
page({isAdmin: true, people: [{name: 'lisi'}, {name: 'zhangsan'}]});
```

下面使用模板字符串来实现该功能

```javascript
function capitalize(string) {
  return string[0].toUpperCase() + string.slice(1);
}

var p = ({people, isAdmin}) => `
  <h2>People</h2>
  <ul>
    ${people.map(person => `
      <li>
        <span>${capitalize(person.name)}</span>
        ${isAdmin ? `<button>Delete </button>` : ''}
      </li>
    `).join('')}
  </ul>
`;

p({isAdmin: true, people: [{name: 'lisi'}, {name: 'zhangsan'}]});
```

怎么样，是不是很简洁！还没完事呢，下满我们再继续优化这段代码。

## 给模板字符串增加辅助方法

```javascript
var helpers = {
  if(condition, thenTpl, elseTpl = '') {
    return condition ? thenTpl : elseTpl;
  },
  registerHelper(name, fn) {
    helpers[name] = fn;
  }
};

helpers.registerHelper('capitalize', (string) => {
  return string[0].toUpperCase() + string.slice(1);
});

var p = ({people, isAdmin}) => `
  <h2>People</h2>
  <ul>
    ${people.map(person => `
      <li>
        <span>${helpers.capitalize(person.name)}</span>
        ${helpers.if(isAdmin, `<button>Delete </button>`)}
      </li>
    `).join('')}
  </ul>
`;
```
这里只演示了将三元表达式逻辑抽离到helper中if函数，还可以自定义helper函数，这样可以把复杂的业务逻辑抽离出来，让模板字符串尽量保持简洁。

## 与Symbol相结合

## 使用微模板来优化代码

## 总结
