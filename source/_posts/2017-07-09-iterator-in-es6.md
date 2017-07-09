---
title: 学习ES6中的迭代器
date: 2017-07-09 12:08:02
tags: ES6
categories: ES6 
---

## 迭代协议

> ECMAScript2015的几个补充，并不是新的内置或语法，而是协议。这些协议可以被任何遵循某些约定的对象来实现。它有两个协议：可迭代协议和迭代器协议。

<!--more-->

### 可迭代协议

用来定义javascript的对象是否可以被迭代，为了变成可迭代对象，该对象必须实现@@iterator方法，意思是这个对象（或者原型链上）必须有一个名字是Symbol.iterator属性。

### 迭代器协议

该迭代器协议定义了一种标准的方式来产生一个有限或无限序列的值。
当一个对象被认为是一个迭代器时，它实现了一个 next() 的方法并且拥有以下含义：
返回一个对象的无参函数，被返回对象拥有两个属性：

1. done boolean
	* 如果迭代器已经经过了被迭代序列时为 true。
	* 如果迭代器可以产生序列中的下一个值，则为 false。
2. value
	 迭代器返回的任何 JavaScript 值。done 为 true 时可省略。

## 自定义可迭代对象

下面代码创建了一个自定义的可迭代对象

```javascript
var customIterator01 = (function() {
  var val = 1;
  return {
    [Symbol.iterator]() {
      return {	
        next() {
          val = val * 2;
          return {done: val >= 10, value: val};
        }
      };
    }
  };
})();
```

使用for of进行迭代

for of 专门是针对可迭代对象的迭代而新增的API，不只是局限于Array。其内部会一直调用iterator的next方法，直到done返回true，此时的value并不会返回，没什么实际的意义了。

```javascript
for(let val of customIterator01) {
  console.log(val); // 2, 4, 8
}
```

使用展开运算符 ...

展开运算符内部也同样使用了迭代协议

```javascript
[...customIterator01]  // [2, 4, 8]
```

## 内置可迭代对象

String, Array, TypedArray, Map 和 Set是所有内置可迭代对象， 因为它们的原型对象都有一个 @@iterator 方法。

来一些例子

```javascript
[..."hello"]                                             // ["h", "e", "l", "l", "o"]
new Map([[1,"a"],[2,"b"],[3,"c"]]).get(2);               // "b"
new WeakMap([[{},"a"],[myObj,"b"],[{},"c"]]).get(myObj); // "b"
new Set([1, 2, 3]).has(3);                               // true
new Set("123").has("2");                                 // true
new WeakSet(function*() {
    yield {};
    yield myObj;
    yield {};
}()).has(myObj);                                         // true
```

## 生成器式的迭代器

```javascript
// 创建一个生成器
function* makeSimpleGenerator(array){
  var nextIndex = 0;
  
  while(nextIndex < array.length){
    yield array[nextIndex++];
  }
}

var gen = makeSimpleGenerator(['yo', 'ya']);

console.log(gen.next().value); // 'yo'
console.log(gen.next().value); // 'ya'
console.log(gen.next().done);  // true

// 迭代该生成器
for(let val of makeSimpleGenerator(["hello", "world"])) {
  console.log(val); // "hello", "world"
}
```
ES6中的Generator后续会专门做详细介绍，这里先简单说一下：

1. generator的定义是在function后加* 或者在函数名前加*，例如function *name(){}。
2. yield 程序运行到此会暂停，直到generator实例调用next，程序会继续运行。
3. yield 和 next之间可以互相传递值。
4. generator它实现了Symbol.itarator和next，因此它是可迭代的，同时也是一个迭代器。



