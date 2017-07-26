---
title: es6-part-three
date: 2017-07-24 23:21:18
tags:
categories:
---

# Set集合与Map集合

## ES5中的Set集合与Map集合

```javascript
var set = Object.create(null);

set.foo = true;

// 检查属性是否存在
if(set.foo){
  // 要执行的代码
}
```

上面是模拟Set集合，而Map集合只是存储的值不同。

## 该解决方案的一些问题

### 非字符串类型的key

```javascript
var map = Object.create(null);
map[5] = "foo";

console.log(map["5"]);   // "foo"
```

```javascript
var map = Object.create(null);
var key1 = {},
  key2 = {};

map[key1] = "foo";

console.log(map[key2]);   // "foo"
```

上面例子对象的属性名都会做类型转化，转化为字符串。其中Object会调用其toString()方法，因此key1和key2都得到"[object Object]"。

## ES6中的Set集合

ES6中的Set类型是一种有序列表（Java中是无序的），其中含有一些相互独立的非重复值，通过Set集合可以快速访问其中的数据，更有效地追踪各种离散值。

### 创建Set集合并添加元素

```javascript
let set = new Set();
set.add(5);
set.add("5");

console.log(set.size);     // 2 
```

Set 集合中，不会对所存值进行强制类型转换。也可以向Set中添加多个对象，则它们之间彼此保持独立：

```javascript
let set = new Set(),
  key1 = {},
  key2 = {};

set.add(key1);
set.add(key2);

console.log(set.size);     // 2
```

如果多次调用add()方法并传入相同的值，后续的调用会被忽略。

```javascript
let set = new Set([1, 2, 2, 3]);
console.log(set.size);     // 3
```

**实际上，Set构造函数可以接受所有可迭代对象作为参数，数组、Set集合、Map集合等。**

### 移除元素

使用delete()可以移除Set集合中的某一个元素，clear()方法可以移除所有元素。

### Set集合的forEach()方法

forEach方法的回调函数接收以下3个参数：

* Set集合中下一次索引的位置
* 与第一个参数一样的值
* 被遍历的Set集合本身

在Set集合的forEach()方法中，第二个参数也与数组的一样，如果需要在回调函数中使用this引用，则可以将它作为第二个参数传入forEach()函数。

### 将Set集合转换为数组

将数组转化为Set集合的过程很简单，只需给Set构造函数传入数组即可；将Set集合再转回数组同样很简单，可以用展开运算符。

```javascript
let set = new Set([1, 2, 3, 3, 4, 5]),
  array = [...set];

console.log(array);     // [1, 2, 3, 4, 5]
```

### Weak Set集合

将对象存储在Set的实例与存储在变量中完全一样，只要Set实例中的引用存在，垃圾回收机制就不能释放该对象的内存空间，于是之前提到的Set类型可以被看作是一个强引用的Set集合。

```javascript
let set = new Set(),
  key = {};

set.add(key);
console.log(set.size);      // 1

// 移除原始引用
key = null;

console.log(set.size);      // 1

// 重新取回原始引用
key = [...set][0];
```
在这个示例中，将变量key设置为null时便清除了对初始对象的引用，但是Set集合却保留了这个引用，仍然可以使用展开运算符将Set集合转换成数组格式并从数组中取出该引用。大部分情况这段代码运行良好，但有时候你会希望当其他引用都不存在时，让Set集合中的这些引用随之消失。例如：在Web页面通过js记录可一些DOM元素，这些元素有可能被另一段脚本移除，而你又不希望自己的代码保留这些DOM元素的最后一个引用。

为了解决这个问题，ES6中引入了另外一个类型：Weak Set集合（弱引用Set集合）。Weak Set集合只存储对象的弱引用，并且不可以存储原始值；集合中的弱引用如果是对象唯一的引用，则会被回收并释放相应内存。

### 创建Weak Set集合

集合支持3个方法：add()、has()和delete()。

```javascript
let set = new WeakSet(),
  key = {};

set.add(key);

console.log(set.has(key));      // true

selt.delete(key);

console.log(set.has(key));      // false

```

也可以使用构造函数来创建WeakSet，其参数不能为任何原始值，如果包含其他非对象值，程序会抛出错误。

### 两种Set的主要区别

**最大的区别是Weak Set保存的是对象值的弱引用。**

* 在Weak Set实例中，如果向add()、has()、delete()方法传入非对象参数都会导致程序出错。
* Weak Set集合不可迭代
* Weak Set集合不暴露任何迭代器，所以无法通过程序本身来检测其中的内容。
* Weak Set集合不支持forEach()方法。
* Weak Set集合不支持size属性。

Weak Set集合的功能看似受限，其实这是为了让它能够正确地处理内存中的数据。总之，如果你只需要跟踪对象引用，你更应该使用Weak Set集合而不是普通的Set集合。


