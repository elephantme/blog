---
layout: post
title:  "探索Javascript中的继承"
date:   2016-07-10
categories: javascript
---

## 1. 原型链

通过原型链可以作为实现继承的主要方法。其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。我们知道每个实例都包括指向原型对象的内部指针，如果我们将该实例直接赋值给另一个原型对象，会发生什么呢？显然此时该原型对象将包括一个指向另一个原型的指针，以此类推下去，就构成了实例与原型的链条。这就是所谓原型链的基本概念。

<!--more-->

其代码大致如下：

```javascript
var SuperClass = function(){
  this.superValue = "super";
};
SuperClass.prototype.getSuperValue = function(){
  return this.superValue;
};

var SubClass = function(){
  this.subValue = "sub";
};
//继承SuperClass
SubClass.prototype = new SuperClass();
SubClass.prototype.getSubValue = function(){
  return this.subValue;
};
var instance = new SubClass();
console.log(instance.getSuperValue()); //super
console.log(instance.getSubValue()); //sub
```

上述代码SubClass继承了SuperClass，是通过将SuperClass的实例赋值给SubClass的原型对象实现的，注意此时SubClass.prototye.constructor也会是SuperClass。因此应该加上`SubClass.prototype.construcotr=SubClass`。这个例子中的实例以及构造函数和原型之间的关系如下图

![javascript prototype inherit](/images/js-prototype-extend.png)

### 1.1 原型链的问题

1. 原型的属性会被所有实例共享
2. 创建子类型实例时，不能向父类型构造函数中传递参数

下面代码举例说明`lans`属性被所有实例共享

```javascript
var SuperClass = function(){
  this.superValue = "super";
  this.lans = ["java","javascript"];
};
SuperClass.prototype.getSuperValue = function(){
  return this.superValue;
};

var SubClass = function(){
  this.subValue = "sub";
};
SubClass.prototype = new SuperClass();
SubClass.prototype.getSubValue = function(){
  return this.subValue;
};
var instance1 = new SubClass();
instance1.lans.push("ruby");
console.log(instance1.lans);  //["java", "javascript", "ruby"]

var instance2 = new SubClass();
console.log(instance2.lans); //["java", "javascript", "ruby"]
```

## 2. 借用构造函数

在解决原型中引用类型被实例共享问题的过程中，一直使用`借用构造函数`的方式。其思想也比较简单，即在子类型构造函数的内部调用超类型构造函数。可以通过call()或者apply()来借用
父类构造函数，同时还可以传递参数。

```javascript
var SuperClass = function(){
  this.lans = ["java","javascript"];
};

var SubClass = function(){
  //继承SuperClass
  SuperClass.call(this);
};

var instance1 = new SubClass();
instance1.lans.push("ruby");
console.log(instance1.lans);  //["java", "javascript", "ruby"]

var instance2 = new SubClass();
console.log(instance2.lans); //["java", "javascript"]
```

传递参数的示例：

```javascript
var SuperClass = function(name){
  this.name = name;
};

var SubClass = function(name){
  SuperClass.call(this, name);
};

var instance1 = new SubClass("zhangsan");
console.log(instance1.name);  //zhangsan

var instance2 = new SubClass("lisi");
console.log(instance2.name); //lisi
```

### 2.1 借用构造函数的问题

无法重用父类中的方法，因此就有了下面提到的组合继承。

## 3. 组合继承

就是把原型链和借用构造函数组合起来实现继承的一种方式，大概代码如下：

```javascript
var SuperClass = function(name){
  this.name = name;
};
SuperClass.prototype.getName = function(){return this.name;}

var SubClass = function(name, age){
  SuperClass.call(this, name);
  this.age = age;
};
//原型对象重新赋值，继承SuperClass的方法
SubClass.prototype = new SuperClass();
SubClass.prototype.constructor = SubClass;
//重写了Object.toString方法
SubClass.prototype.toString = function(){
  return "[name:"+this.name+",age:"+this.age+"]";
}

var instance1 = new SubClass("zhangsan", 15);
console.log(instance1.getName()); //zhangsan
console.log(instance1.toString());  //[name:zhangsan,age:15]

var instance2 = new SubClass("lisi", 20);
console.log(instance2.toString()); //[name:lisi,age:20]
```


