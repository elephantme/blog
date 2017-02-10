---
layout: post
title:  "数据结构"
date:   2016-07-21
categories: data_structure
tags: data_structure
---

介绍常见的数据结构，如集合、散列表、队列、栈、链表等，并用js来实现。

<!--more-->

## 集合

集合是由一组无序且唯一的项组成。这个数据结构使用了与有限集合相同的数学概念，比如：自然数集合N={0,1,2,...}

集合会有一些运算，在本文也实现一下。

另外ES6中已经提出Set概念，[API点击这里](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)

```javascript
class Set{
  constructor(){
    this.items = {}
  }

  add(value){
    if(!this.has(value)){
      this.items[value] = value;
      return true;
    }
    return false;
  }
  remove(value){
    if(this.has(value)){
      delete this.items[value];
      return true;
    }
    return false;
  }
  has(value){
    return this.items.hasOwnProperty(value);
  }
  clear(){
    this.items = {};
  }
  size(){
    return Object.keys(this.items).length;
  }
  values(){
    let result = [];
    for(var key in this.items){
      result.push(this.items[key]);
    }
    return result;
  }

  // 并集
  union(otherSet){
    let unionSet = new Set();
    let values = this.values();
    values.forEach((item) => {unionSet.add(item);});

    values = otherSet.values();
    values.forEach((item) => {unionSet.add(item);});

    return unionSet;
  }

  // 交集
  intersection(otherSet){
    let intersectionSet = new Set();

    this.values().forEach((item) => {
      if(otherSet.has(item)){
        intersectionSet.add(item);
      }
    });

    return intersectionSet;
  }

  // 差集
  difference(otherSet){
    let differenceSet = new Set();

    this.values().forEach((item) => {
      if(!otherSet.has(item)){
        differenceSet.add(item);
      }
    });

    return differenceSet;
  }

  // 是否是其它集合的子集
  subset(otherSet){
    if(this.size() > otherSet.size()){
      return false;
    }

    return this.values().every((item) => {
      return otherSet.has(item);
    });

  }
}
```

## 字典

字典与集合的区别就是，字典存储的键值对，它也称作映射。

```javascript
class Dictionary{
  constructor(){
    this.items = {};
  }

  set(key, value){this.items[key] = value;}

  get(key){return this.items[key];}

  remove(key){delete this.items[key];}

  has(key){return key in this.items;}

  clear(){this.items = {};}

  size(){return Object.keys(this.items).length;}

  keys(){return Object.keys(this.items);}

  values(){
    let relt = [];
    this.keys().forEach((key) => {relt.push(this.items[key])});
    return relt;
  }
}
```

## 散列表

散列表实现是将key通过散列算法计算得到一个散列值，这样根据散列值直接查找效率要高，不需要像字典那样需要进行迭代。有时不同的key经过散列算法计算会得到相同的散列值，这种冲突解决的方案一般有`链表法`、`线性探测`。

链表法：为散列表的每一个位置创建一个链表并将元素存储在里面。他是解决冲突最简单的方法，但是他在HashTable实例之外需要额外的存储空间。Java中的HashMap就是采用链表法来解决hash冲突的。

线性探测：当向某个位置加入一个新元素的时候，如果索引为index的位置被占据了，就尝试index+1的位置，以此类推。

### 链表法实现

```javascript
LinkedHashTable = (function(){
  var KV = function(key, value){
    this.key = key;
    this.value = value;
  };

  var LinkedHashTable = function(){
    this.items = [];
  };

  //hashcode算法
  var hashcode = function(key){
    var hash = 0;
    for(var i = 0; i < key.length; i++){
      hash += key.charCodeAt(i);
    }
    return hash % 37;
  };

  LinkedHashTable.prototype = {
    contructor: LinkedHashTable,
    put: function(key, value){
      var index = hashcode(key);
      
      //每个位置用linkedList来进行存储      
      if(this.items[index] === undefined){
        this.items[index] = new LinkedList();
      }

      this.items[index].append(new KV(key, value));
    },
    get: function(key){
      var index = hashcode(key),
        linkedList = this.items[index];

      if(linkedList !== undefined){
        var current = linkedList.getHead();
        // 从链表头开始查找
        while(current){
          if(key === current.element.key){
            return current.element.value;
          }
          current = current.next;
        }
      }

      return null;
    },
    remove: function(key){
      var index = hashcode(key),
        linkedList = this.items[index];

      if(linkedList !== undefined){
        var current = linkedList.getHead();
        // 从链表头开始查找
        while(current){
          if(key === current.element.key){
            linkedList.remove(current.element);
            return true;
          }
          current = current.next;
        }
      }

      return false;
    }
  };

  return LinkedHashTable;
})();
```

### 线性探测实现

```javascript
var LinearHashTable = (function(){
  function LinearHashTable(){
    this.items = [];
  }

  var KV = function(key, value){
    this.key = key;
    this.value = value;
  }

  var hashcode = function(key){
    var hash = 0;
    for(var i = 0; i < key.length; i++){
      hash += key.charCodeAt(i);
    }
    return hash % 37;
  }

  LinearHashTable.prototype = {
    constructor: LinearHashTable,
    put: function(key, value){
      var hash = hashcode(key);

      if(this.items[hash] === undefined){
        this.items[hash] = new KV(key, value);
      }else{
        while(this.items[++hash] !== undefined){}
        this.items[hash] = new KV(key, value);  
      }
    },
    get: function(key){
      var hash = hashcode(key);

      var kv = this.items[hash];
      if(kv !== undefined){
        if(kv.key === key){
          return kv.value;
        }
        // 继续向后探测的条件：key不相等。第一个条件是为了跳过中间已删除的元素
        var index = ++hash;
        while(this.items[index] === undefined || this.items[index].key !== key 
          && index >= this.items.length){
          index++;
        }
        // 有可能key就不存在
        if(this.items[index].key === key){
          return this.items[index].value;
        }
      }
      return undefined;
    },
    remove: function(key){
      var hash = hashcode(key);

      var kv = this.items[hash];
      if(kv !== undefined){
        if(kv.key === key){
          this.items[hash] = undefined;
          return true;
        }

        var index = ++hash;
        while(this.items[index] === undefined || this.items[index].key !== key
          && index >= this.items.length){
          index++;
        }
        // 有可能key就不存在
        if(this.items[index].key === key){
          this.items[index] = undefined;
          return true;
        }
      }
      return false;
    }
  };

  return LinearHashTable;
})();
```

## 栈

栈是一种遵循后进先出（LIFO）原则的有序集合。新增加的或待删除的都保存在栈的末尾，称作栈顶，另一端就叫栈底。

如下代码实现了一些栈的常用操作

```javascript
var Stack = (function(){
  var Stack = function(){
    this.items = [];
  };

  Stack.prototype = {
    //添加一个或几个元素到栈顶
    push: function(element){
      this.items.push(element);
    },
    //移除栈顶的元素，同时返回该元素
    pop: function(){
      return this.items.pop();
    },
    //返回栈顶的元素，不会对栈做任何修改
    peek: function(){
      return this.size() ? this.items[this.size() - 1] : null;
    },
    //如果栈里没有任何元素就返回true，否则返回false
    isEmpty: function(){
      return this.items.length === 0;
    },
    //移除栈里的所有元素
    clear: function(){
      this.items = [];
    },
    //返回栈里元素的个数
    size: function(){
      return this.items.length;
    }
  }

  return Stack;
})();
```

## 队列

队列是遵循FIFO（First in first out，先进先出）

用ES6中class语法来实现该数据结构

```javascript
class Queue{
  constructor(){
    this.items = [];
  }

  // 向队列尾部添加元素
  enqueue(element){
    this.items.push(element);
  }
  // 移除队列的第一项，并且返回该元素
  dequeue(){
    return this.items.shift();
  }
  // 查询队列中的第一个元素
  front(){
    return this.size() ? this.items[0] : null;
  }
  // 队列是否为空
  isEmpty(){
    return this.size() === 0;
  }
  // 队列中元素的个数
  size(){
    return this.items.length;
  }
}
```

然而在现实生活中，时常会有优先队列这种现象，如机场登机顺序，头等舱和商务舱乘客的优先级高于经济舱乘客。

只需要重写入队列的方法就可以实现优先队列

```javascript
class PriorityElement{
  constructor(element, priority){
    this.element = element;
    this.priority = priority;
  }
}

// 继承普通队列
class PriorityQueue extends Queue{
  // 重写入队列的方法，按优先级插入
  enqueue(element, priority){
    let priorityElement = new PriorityElement(element, priority);
    if(this.size() === 0){
      this.items.push(priorityElement);
    }else{
      let inserted = false;
      this.items.every((item,index) => {
        if(priority < item.priority){
          this.items.splice(index,0,priorityElement);
          inserted = true;
          return false;
        }
      });
      if(!inserted){
        this.items.push(priorityElement);
      }
    }
  }

}
```

## 链表

链表存储有序的元素集合，但不同于数组，列表中的每个元素在内存中并不是连续放置的。每个元素有一个存储元素本身的节点和一个指向下一个元素的引用组成。

相对于数组，他的优点是添加或移除元素不需要移动其它元素。数组可以直接访问任何位置的任何元素，而想要访问链表中的元素，需要从起点开始迭代列表知道找到该元素为止。

单项链表：

```javascript
class Node{
  constructor(element){
    this.element = element;
    this.next = null;
  }
}

class LinkedList{
  constructor(){
    this.head = null;
    this.length = 0;
  }

  // 向链表尾部添加一个新的项
  append(element){
    let node = new Node(element),
      current;

    if(this.head === null){
      this.head = node;
    }else{
      // 准备从头部开始迭代
      current = this.head;
      // 只要next不为空就一直迭代下去
      while(current.next){
        current = current.next;
      }
      // 到这里已经找到最后一项了，将node赋值给next
      current.next = node;
    }

    this.length++;
  }

  // 向链表的特定位置插入一个新的项
  insert(position, element){
    if(position >=0 && position <= this.length){
      let node = new Node(element),
        index = 0,
        current = this.head,
        previous;

      if(position === 0){
        node.next = current;
        this.head = node;
      }else{
        // 迭代找到要插入元素的问题
        while(index++ < position){
          previous = current;
          current = current.next;
        }
        // 指针赋值
        previous.next = node;
        node.next = current;
      }
      this.length++;
      return true;
    }else{
      return false;
    }
  }

  // 从链表的特定位置移除一项
  removeAt(position){
    if(position >= 0 && position < this.length){
      let current = this.head,
        previous, //前一个
        index = 0; //计数

      // 移除第一个元素
      if(position === 0){
        this.head = current.next;
      }else{
        while(index++ < position){
          // 上一个元素存储起来
          previous = current;
          // 当前元素指向下一个元素进行迭代
          current = current.next;
        }
        // 将上一个元素的next指向下一个元素，就把当前元素删除了
        previous.next = current.next;
      }
      this.length--;
      return current.element;
    }else{
      return null;
    }
  }

  // 从链表中移除一项
  remove(element){
    var index = this.indexOf(element);
    return this.removeAt(index);
  }

  // 返回元素在链表中的索引。如果没找到则返回-1
  indexOf(element){
    let current = this.head,
      index = 0;
    while(current){
      if(element === current.element){
        return index;
      }
      index++;
      current = current.next;
    }
    return -1;
  }

  // 如果链表没有任何元素返回true，否则返回false
  isEmpty(){
    return this.length === 0;
  }

  // 链表中元素个数
  size(){
    return this.length;
  }

  toString(){
    let arr = [],
      current = this.head;

    while(current){
      arr.push(current.element);
      current = current.next;
    }

    return arr.join(",");
  }
}
```

## 双向链表

双向链表即包括指向下一个元素的指针，也包括指向上一个元素的指针。

```javascript
class Node{
  constructor(element){
    this.element = element;
    this.next = null;
    this.prev = null;
  }
}

class DoublyLinkedList{
  constructor(){
    this.head = null;
    this.tail = null;
    this.length = 0;
  }

  insert(position,element){
    if(position >= 0 && position <= this.length){
      let node = new Node(element),
        current = this.head,
        previous,
        index = 0;

      // 添加头部 
      if(position === 0){
        // 第一次添加
        if(!this.head){
          this.head = node;
          this.tail = node;
        }else{
          node.next = current;
          current.prev = node;
          this.head = node;
        }
        // 添加尾部
      }else if(position === this.length){
        current = this.tail;
        current.next = node;
        node.prev = current;
        this.tail = node;
      }else{
        // 查找要插入的位置
        while(index++ < position){
          previous = current;
          current = current.next;
        }
        // 通过next建立好链接
        previous.next = node;
        node.next = current;

        // 再关联prev
        node.prev = previous;
        current.prev = node;
      }

      // 长度增加
      this.length++;

      return true;
    }

    return false;
  }

  removeAt(position){
    if(position >= 0 && position < this.length){
      let current = this.head,
        previous,
        index = 0;

      // 移除头
      if(position === 0){
        this.head = current.next;
        // 只有一个元素的情况
        if(length === 1){
          this.tail = null;
        }else{
          this.head.prev = null;
        }
        // 移除尾
      }else if(position === this.length - 1){
        current = this.tail;
        this.tail = current.prev;
        this.tail.next = null;
      }else{
        // 查找要移除的位置
        while(index++ < position){
          previous = current;
          current = current.next;
        }

        // 前一个元素next指向下一个元素
        previous.next = current.next;
        // 下一个元素prev指向前一个元素
        current.next.prev = previous;
      }

      this.length--;

      return current.element;
    }

    return null;
  }

  toString(){
    let arr = [],
      current = this.head;

    while(current){
      arr.push(current.element);
      current = current.next;
    }

    return arr.join(",");
  }
}
```

