---
layout: post
title:  "JavaScript中的事件机制"
date:   2016-07-20
categories: javascript
tags: javascript
---

## 1. 事件发展的历史

### 1.1 行内事件（Inline Events）

这是浏览器最早支持的写法，现在都不这么写了。

```html
<div onclick="fn()">Click</div>
```

<!--more-->

### 1.2 DOM0 Events

```html
<div id="ele">Click me</div>
<script>
var ele = document.getElementById('ele');
ele.onclick = function(){
  console.log(this.id);
};
</script>
```

1. 绝大多数浏览器都支持该写法
2. event handler中this就是触发事件的元素（IE有些差别，this指的是全局对象）。
3. 只能绑定一个事件，因为是赋值操作。

### 1.3 DOM2 Events

该机制可以支持绑定多个事件，但是浏览器API会有些不同。

```javascript
element.addEventListener(eventType, handler, isCapture);
element.attachEvent(eventType, handler);
```

### 1.4 事件兼容的写法

```javascript
function addEvent(element, type, handler){
  if(element.addEventListener){
    element.addEventListener(type, handler, false);
  }else if(element.attachEvent){
    element.attachEvent("on"+type, handler);
  }else{
    element["on"+type] = handler;
  }
}
```

## 2. 事件的传播

事件的传播方式有两种`捕获`和`冒泡`，提出这两种机制的浏览器厂商分别是Netscape和Microsoft。捕获可以理解事件是由外向内传播，而冒泡刚好相反。后来W3C制定了一套事件的传播机制，先是捕获阶段，然后到达目标元素，最后是冒泡阶段。

下面举例说明一下

```html
<div id="outer">
  This is outer.
  <div id="middle">This is middle.
    <div id="inner">This is inner.</div>
  </div>
</div>
<script>
var outer = document.getElementById('outer');
var middle = document.getElementById('middle');
var inner = document.getElementById('inner');

outer.addEventListener("click", function(e){
  console.log("click outer")
},false);

middle.addEventListener("click", function(e){
  console.log("click middle");
},true);

inner.addEventListener("click", function(e){
  console.log("click inner");
},false);
</script>
```

如果用户单击了inner，事件传播过程如下：

1. 捕获阶段。寻找inner父级元素是否使用捕获的方式来定义click事件，结果找到middle，所以先输出了click middle。
2. 目标阶段。inner自己输出click inner。
3. 冒泡阶段。寻找inner父级元素是否使用冒泡的方式定义了click事件，结果找到outer，所以最后输出了click outer。

## 3. Event handler兼容性代码

```javascript
//获取事件对象
var getEvent = function(event){
  //标准实现返回event，而IE中事件对象是存放在window中
  return event || window.event;
};

//获取元素
var getTarget = function(event){
  var event = getEvent(event);
  //标准实现返回target，而IE下用srcElement
  return event.target || event.srcElement;
};

//阻止捕获
var preventDefault = function(event){
  var event = getEvent(event);
  if(event.preventDefault){
    event.preventDefault();
  }else{
    //IE
    event.returnValue = false;
  }
};

//阻止冒泡
var stopPropagation = function(event){
  var event = getEvent(event);
  if(event.stopPropagation){
    event.stopPropagation();
  }else{
    //IE
    event.cancelBubble = true;
  }
};
```




