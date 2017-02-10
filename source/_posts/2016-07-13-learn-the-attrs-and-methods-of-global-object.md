---
layout: post
title:  "介绍js全局对象的常量属性&方法"
date:   2016-07-13
categories: javascript
tags: javascript
---

## 1. 常量属性

### 1.1 Infinity

是一个数值，表示无群大。

### 1.2 NaN

1. 该属性的初始值就是NaN，和Number.NaN的值一样。
2. 等号运算符（==和===）不能用来判断一个是是否是NaN。必须使用Number.isNaN()或isNaN()函数。

<!--more-->

```javascript
NaN === NaN; //false
Number.NaN === NaN; //false
isNaN(NaN); //true
isNaN(Number.NaN); //true
```

### 1.3 undefined

1. window下的一个属性，并且该属性的属性值也为undefined。
2. 一个未初始化的变量的值为undefined。
3. 一个没有传入实参的形参变量的值为undefined。
4. 一个函数什么都没返回，则该函数默认返回undefined。
5. 可以用严格相等（===）运算符来判断一个值是否是undefined。
6. 另外还可以使用typeof来判断，变量没有申明的情况下，也只能用它来判断了。

### 1.4 null（字面量）

1. null是一个字面量，表示空值，并不是全局变量的属性。
2. null 与 undefined 的不同点

```javascript
typeof null;      //object
typeof undefined; //undefined
null === undefined; //false
null == undefined;  //true
```

## 2. 方法属性

### 2.1 eval(string)

1. string为js的表达式，或者声明。
2. 如果参数是表达式，则会求值，如果是一个或者多个声明，那么eval会执行声明。
3. 参数如果不是字符串，则会将参数原封不动的返回。

### 2.2 uneval(object)

函数创建一个代表对象的源代码的字符串。

### 2.3 isFinite(testValue)

1. 用来判断传入的参数是否是一个有限数值。
2. 如果参数是 NaN，正无穷大或者负无穷大，会返回false，其他返回 true

### 2.4 isNaN(testValue)

1. 用来判断一个值是否是NaN。
2. 如果isNaN函数的参数不是Number类型, isNaN()会首先尝试将这个参数转换为数值。

### 2.5 parseFloat(string)

1. 方法将参数中指定的字符串解析成为一个浮点数字并返回。
2. 在解析过程中遇到了正负号(+或-),数字(0-9),小数点,或者科学记数法中的指数(e或E)以外的字符,则它会忽略该字符以及之后的所有字符,返回当前已经解析到的浮点数。
3. 如果参数字符串的第一个字符不能被解析成为数字,则parseFloat返回NaN。

### 2.6 parseInt(string,radix)

1. 将给定的字符串以指定基数（radix/base）解析成为整数。
2. 如果 parseInt 遇到了不属于radix参数所指定的基数中的字符那么该字符和其后的字符都将被忽略。接着返回已经解析的整数部分。parseInt 将截取整数部分。开头和结尾的空白符允许存在，会被忽略。

下面例子都返回15

```javascript
parseInt("F", 16); 
parseInt("17", 8);
parseInt("15", 10);
parseInt(15.99, 10);
parseInt("FXX123", 16);
parseInt("1111", 2);
parseInt("15*3", 10);
parseInt("12", 13);
```

### 2.7 decodeURI(encodedURI)

用于解码由 encodeURI 方法或者其它类似方法编码的统一资源标识符（URI）

### 2.8 decodeURIComponent(encodedURI)

用于解码由 encodeURIComponent 方法或者其它类似方法编码的部分统一资源标识符（URI）。

### 2.9 encodeURI(URI)

1. 是对统一资源标识符（URI）进行编码的方法。
2. encodeURI 自身无法产生能适用于HTTP GET 或 POST 请求的URI，例如对于 XMLHTTPRequests, 因为 "&", "+", 和 "=" 不会被编码，然而在 GET 和 POST 请求中它们是特殊字符。然而encodeURIComponent这个方法会对这些字符编码。

### 2.10 encodeURIComponent(URI)

### 2.11 escape() 已废弃

生成新的由十六进制转移序列替换的字符串. 使用 encodeURI 或 encodeURIComponent 代替

### 2.12 unescapt() 已废弃


