---
title: ES6之生成器(Generator)
date: 2017-07-11 23:09:32
tags: ES6, Generator
keywords: Generator, 生成器, ES6
categories: ES6
---

> 本文主要介绍Generator的基础知识点、怎么与异步方法及Promise结合使用。

## 基础知识

### function*

这种方式会定义一个生成器函数，它返回一个Generator对象，而Generator对象遵循可迭代协议和迭代器协议。

生成器函数在中途的执行过程中可以退出，后面又能重新进入继续执行。而在函数内部定义的变量状态不会受到中途退出的影响。

<!--more-->

调用一个生成器函数不会马上执行函数体的语句，而是得到一个该生成器的迭代器（iterator）对象。当这个迭代器的next()方法首次被调用时，其内部的语句才开始执行，遇到yield语句会停止（该yield语句定义了迭代器要返回的值，存放在value属性中），或者被yield*委派到另一个生成器函数。next()方法返回一个对象，其包含两个属性：value和done，value表示本次yield表达式的返回值，done为布尔类型，表示生成器是否已经产出了它最后的值，即生成器函数是否已经返回。调用next()方法时，如果传递了参数，那么这个参数会取代生成器函数中对应执行位置的yield表达式（整个表达式被这个值替换）。这种机制意味着yield和next之间可以互相传递数据。

当在生成器函数中显示return，会导致生成器立即变为完成状态，即调用next（）方法返回的对象的done为true。如果return了一个值，那么这个值会作为next()方法返回的value值。

```javascript
function *foo(x) {
  var y = x * (yield);
  return y;
}

var it = foo(5);
var res1 = it.next();  // res1: {done: false, value: undefined}
var res = it.next(6);  // res: {done: true, value: 30}
```
首先，传入5给x同时创建了迭代器it。紧接着调用next()，代码开始执行，然后遇到yield程序停止，该yield表达式值为undefined，所以第一个next()执行的返回值中value为undefined。接着it.next(6)执行，参数6取代yield表达式，因此y=5*6,后面遇到return，生成器状态变为true，并且把返回值30，设置为next返回值中value属性的值。

我们将上面的例子做一个小小改动，如下：

```javascript
function *foo(x) {
  var y = x * (yield 5);
  return y;
}

var it = foo(5);
var res1 = it.next();                // res1: {done: false, value: 5}
var res = it.next(res1.value + 1);   // res: {done: true, value: 30}
```
不同之处在于yield表达式的值为5，因此第一个next()返回值中value也为5。这个例子就实现了yield给next传递值，然后next又替换yield表达式的值。

### yield

yield关键字用来暂停和恢复一个生成器函数。

yield关键字使生成器函数执行暂停，yield关键字后面的表达式的值返回给生成器的调用者。它可以被认为是一个基于生成器的版本的return关键字。

yield关键字实际返回一个IteratorResult对象，它有两个属性：value和done。value属性是对yield表达式求值的结果，而done是false，表示生成器函数尚未完全完成。

一旦在yield expression处暂停，除非外部调用生成器的next()方法，否则生成器的代码将不能继续执行。这使得可以对生成器的执行以及渐进式的返回值进行直接控制。

### yield*

yield* 表达式用于委托给另一个generator或可迭代对象。

yield* 表达式迭代操作数，并yield它返回的每个值。

yield* 表达式本身的值是当迭代器关闭时返回的值(即，当done时为true)。

## 异步迭代生成器

下面简单写了个异步代码：

```javascript
function foo(id, cb){
  setTimeout(function() {
    let random = Math.random() * 10;
    if(random < 5) {
      cb.call(null, {error: 'some error occured'}, null);
    }else{
      cb.call(null, null, {content: "value"});
    }
  });
}

foo("12", function(err, content){
  if(err){
    console.error(err);
  }else{
    console.log(content);
  }
});
```

通过生成器可以这样实现：

```javascript
let it = null;
function foo(id){
  setTimeout(function() {
    let random = Math.random() * 10;
    if(random < 5) {
      it.throw({error: 'some error occured'});
    }else{
      it.next({content: "value"});
    }
  });
}

function *main() {
  try{
    let rs = yield foo(1);
    console.log(rs);
  }catch(err){
    console.error(err);
  }
}

it = main();
it.next();
```

这段代码看起来要比回调代码多很多，但最主要的优势是以同步的思维来编写异步代码！请看`let rs = yield foo(1);`这句代码可以从异步代码中得到返回值，回调的方式是不能这样做的。这是巨大的进步！对于无法以顺序同步的，符合我们大脑思考模式的方式表达异步这个问题，这是一个近乎完美的解决方案。

## 生成器+Promise

ES6中最完美的世界就是生成器和Promise的结合！

先来一发Promise例子

```javascript
function foo(id){
  return new Promise((resolve, reject) => {
    setTimeout(function() {
      let random = Math.random() * 10;
      if(random < 5) {
        reject({error: 'some error occured'});
      }else{
        resolve({content: "value"});
      }
    });
  });
}

foo(1).then(data => {
  console.log(data);
}, error => {
  console.error(error);
});
```

使用生成器来实现该功能，代码如下：

```javascript
// function foo 同上
function *main() {
  try{
    let rs = yield foo(1);
    console.log(rs);
  }catch(err){
    console.error(err);
  }
}

var it = main();
var p = it.next().value;
p.then(
  data => it.next(data),
  err => it.throw(err)
)
```

注意到了没，main生成器和异步例子中的是完全一致的！因为yield表达式隐藏了实现的细节。

生成器+Promise结合的核心思路是：**yield出来一个Promise，然后通过这个Promise来控制生成器的迭代器**。

还有一个比较重要的是，Promise驱动的生成器，可以封装出一个通用的库，这样就不至于每次都手动干重复的事情。

比较简单的实现：

```javascript
function run(gen){
  var args = [].slice.call(arguments, 1), it;

  // 在当前上下文初始化生成器
  it = gen.apply(this, args);

  return (function handleNext(value) {
    var next = it.next(value);
    if(next.done){
      return next.value;
    }else{
      return next.value.then(handleNext, err => it.throw(err));
    }
  })();
}

// 则main生成器直接使用 run(main) 来执行就可以了。
```

其实在ES7中已经将这部分的逻辑抽象出来了，请看代码：

```javascript
async function main() {
  try{
    let rs = await foo(1);
    console.log(rs);
  }catch(err){
    console.error(err);
  }
}

main();
```

async、await的实现原理就类似生成器+Promise+run。如果你await了一个Promise，async函数就会自动获知要做什么，它会暂停这个函数（就像生成器一样），直到Promise决议。

是不是顿时感觉世界越来越美好了~

本文中生成器+Promise的例子非常简单，实际工作中Promise的使用场景往往是比较复杂的，有机会再单独来介绍吧。



