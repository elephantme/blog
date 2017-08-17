---
layout: post
title:  "总结下闭包"
date:   2016-07-10
categories: 
tags: 
---

当在函数内部定义了其他函数时，就创建了闭包。闭包有权访问包含函数内部的所有变量，原理如下：

1. 在执行环境中，闭包的作用域链包含着他自己的作用域、包含函数的作用域和全局作用域。
2. 通常，函数作用域及其所有变量会在函数执行结束后被销毁。
3. 但是，当函数返回了一个闭包时，这个函数的作用域将会一直在内存中保存到闭包不存在为止。

<!--more-->

## 1. 使用闭包模仿块级作用域

1. 创建并立即调用一个函数，这样既可以执行其中的代码，又不会在内存中留下对该函数的引用。
2. 结果就是函数内部的所有变量都会被立即销毁——除非将某些变量赋值给了外部作用域中的变量。

## 2. 使用闭包在对象中创建私有变量

1. 即使javascript中没有正式的私有对象属性的概念，但可以使用闭包来实现公有方法，而通过公有方法可以访问在包含作用域中定义的变量。
2. 有权访问私有变量的公有方法叫做特权方法。
3. 可以使用构造函数模式、原型模式来实现自定义类型的特权方法
4. 也可以使用模块模式、增强的模块模式来实现单例的特权方法。

构造函数模式示例代码

```javascript
function MyObject(){
  var privateVar = 10;
  function privateFn(){return false;}

  this.publicMethod = function(){
    privateVar++;
    return privateFn();
  };
}
```

原型模式示例代码

```javascript
(function(){
  //私有变量和私有方法
  var privateVar = 10;
  function privateFn = function(return false;)

  //构造函数
  MyObject = function(){}

  //公有/特权方法
  MyObject.prototype.publicMethod = function(){
    privateVar++;
    return privateFn();
  };
})();
```

模块模式

```javascript
var singleton = function(){
  //私有变量和私有方法
  var privateVar = 10;
  function privateFn = function(return false;)

  return {
    publicProperty: true,
    publicMethod: function(){
      privateVar++;
      return privateFn();
    }
  };
}();
```

增强模块模式

这种增强的代码适合那些单例必须是某种类型的实例，同时还必须添加一些属性和发放对其加以增强的情况。

```javascript
var singleton = function(){
  //私有变量和私有方法
  var privateVar = 10;
  function privateFn = function(return false;)
  
  //创建对象
  var object = new CustomeType();
  object.publicProperty = true;
  object.publicMethod = function(){
    privateVar++;
    return privateFn();
  }
  
  //返回这个对象
  return object;
}();
```
