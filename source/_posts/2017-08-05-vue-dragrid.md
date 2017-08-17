---
title: 学习dashboard中网格拖拽效果的实现
date: 2017-08-05 11:37:18
tags: 
categories:
---

> 本文对[田老师](https://github.com/tm-roamer)写的[flowgrid](https://github.com/tm-roamer/flowgrid)插件做详细的讲解，为此特地写了一个小项目，暂且就叫[vue-dragrid](https://github.com/elephantme/vue-dragrid)，预览效果[点击这里](https://elephantme.github.io/vue-dragrid/)，下面会针对一个个commit来进行讲解，所有commit[请看这里](https://github.com/elephantme/vue-dragrid/commits/master)。

<!-- More -->

## 准备工作

1. 先clone项目到本地。
2. `git reset --hard commit`命令可以使当前head指向某个commit。 

## 完成html的基本布局

点击复制按钮来复制整个commit id。然后在项目根路径下运行`git reset`。用浏览器打开index.html来预览效果，该插件的html主要结果如下：

```html
<!-- 节点容器 -->
<div class="dragrid">
  <!-- 可拖拽的节点，使用translate控制位移 -->
  <div class="dragrid-item" style="transform: translate(0px, 0px)">
    <!-- 通过slot可以插入动态内容 -->
    <div class="dragrid-item-content">
      
    </div>
    <!-- 拖拽句柄 -->
    <div class="dragrid-drag-bar"></div>
    <!-- 缩放句柄 -->
    <div class="dragrid-resize-bar"></div>
  </div>
</div>
```

## 使用vue完成nodes简单排版

先切换commit，安装需要的包，运行如下命令：

```shell
git reset --hard 83842ea107e7d819761f25bf06bfc545102b2944
npm install
<!-- 启动，端口为7777，在package.json中可以修改 -->
npm start 
```

这一步一个是搭建环境，这个直接看webpack.config.js配置文件就可以了。

另一个就是节点的排版（layout），主要思路是把节点容器看成一个网格，每个节点就可以通过横坐标(x)和纵坐标(y)来控制节点的位置，左上角坐标为(0, 0)；通过宽(w)和高(h)来控制节点大小；每个节点还必须有一个唯一的id。这样节点node的数据结构就为：

```json
{
  id: "uuid",
  x: 0,
  y: 0,
  w: 6,
  h: 8
}
```

其中w和h的值为所占网格的格数，例如容器是24格，且宽度为960px，每格宽度就为40px，则上面节点渲染为240px * 320px, 且在容器左上角。

来看一下dragrid.vue与之对应的逻辑：

```javascript
computed: {
  cfg() {
    let cfg = Object.assign({}, config);
    cfg.cellW = Math.floor(this.containerWidth / cfg.col);
    cfg.cellH = cfg.cellW; // 1:1
    return cfg;
  }
},
methods: {
  getStyle(node) {
    return {
      width: node.w * this.cfg.cellW + 'px',
      height: node.h * this.cfg.cellH + 'px',
      transform: "translate("+ node.x * this.cfg.cellW +"px, "+ node.y * this.cfg.cellH +"px)"
    };
  }
}
```

其中cellW、cellH为每个格子的宽和高，这样计算节点的宽和高及位移就很容易了。

## 完成单个节点的拖拽

### 拖拽事件

1. 使用mousedown、mousemove、mouseup来实现拖拽。
2. 这些事件绑定在document上，只需要绑定一次就可以。

执行流程大致如下：

鼠标在拖拽句柄上按下，`onMouseDown`方法触发，在eventHandler中存储一些值之后，鼠标移动则触发`onMouseMove`方法，第一次进入时`eventHandler.drag`为false，其中isDrag方法会根据位移来判断是否是拖拽行为（横向或纵向移动5像素），如果是拖拽行为，则将drag属性设置为true，同时执行`dragdrop.dragStart`方法（一次拖拽行为只会执行一次），之后鼠标继续移动，则就开始执行`dragdrop.drag`方法了。最后鼠标松开后，会执行`onMouseUp`方法，将一些状态重置回初始状态，同时执行`dragdrop.dragEnd`方法。

### 拖拽节点

拖拽节点的逻辑都封装在dragdrop.js这个文件里，主要方法为`dragStart`、`drag`、`dragEnd`。

#### dragStart

在一次拖拽行为中，该方法只执行一次，因此适合做一些初始化工作，此时代码如下：

```javascript
dragStart(el, offsetX, offsetY) {
  // 要拖拽的节点
  const dragNode = utils.searchUp(el, 'dragrid-item');
  // 容器
  const dragContainer = utils.searchUp(el, 'dragrid');
  // 拖拽实例
  const instance = cache.get(dragContainer.getAttribute('name'));
  // 拖拽节点
  const dragdrop = dragContainer.querySelector('.dragrid-dragdrop');
  // 拖拽节点id
  const dragNodeId = dragNode.getAttribute('dg-id');

  // 设置拖拽节点
  dragdrop.setAttribute('style', dragNode.getAttribute('style'));
  dragdrop.innerHTML = dragNode.innerHTML;
  instance.current = dragNodeId;

  const offset = utils.getOffset(el, dragNode, {offsetX, offsetY});
  // 容器偏移
  const containerOffset = dragContainer.getBoundingClientRect();

  // 缓存数据
  this.offsetX = offset.offsetX;
  this.offsetY = offset.offsetY;
  this.dragrid = instance;
  this.dragElement = dragdrop;
  this.dragContainer = dragContainer;
  this.containerOffset = containerOffset;
}
```

1. 参数el为拖拽句柄元素，offsetX为鼠标距离拖拽句柄的横向偏移，offsetY为鼠标距离拖拽句柄的纵向偏移。
2. 通过el可以向上递归查找到拖拽节点（dragNode），及拖拽容器（dragContainer）。
3. dragdrop元素是真正鼠标控制拖拽的节点，同时与之对应的布局节点会变为占位节点（placeholder），视觉上显示为阴影效果。
4. 设置拖拽节点其实就将点击的dragNode的innerHTML设置到dragdrop中，同时将样式也应用过去。
5. 拖拽实例，其实就是dragrid.vue实例，它在created钩子函数中将其实例缓存到cache中，在这里根据name就可以从cache中得到该实例，从而可以调用该实例中的方法了。
6. `instance.current = dragNodeId;`设置之后，dragdrop节点及placeholder节点的样式就应用了。
7. 缓存数据中的offsetX、offsetY是拖拽句柄相对于节点左上角的偏移。

#### drag

发生拖拽行为之后，鼠标move都会执行该方法，通过不断更新拖拽节点的样式来是节点发生移动效果。

```javascript
drag(event) {
  const pageX = event.pageX, pageY = event.pageY;

  const x = pageX - this.containerOffset.left - this.offsetX,
        y = pageY - this.containerOffset.top - this.offsetY;

  this.dragElement.style.cssText += ';transform:translate('+ x +'px, '+ y +'px)';
}
```

主要是计算节点相对于容器的偏移：鼠标距离页面距离-容器偏移-鼠标距离拽节点距离就为节点距离容器的距离。

#### dragEnd

主要是重置状态。逻辑比较简单，就不再细说了。

到这里已经单个节点已经可以跟随鼠标进行移动了。

## 使placeholder可以跟随拖拽节点运动

本节是要讲占位节点(placeholder阴影部分)跟随拖拽节点一起移动。主要思路是：

1. 通过拖拽节点距离容器的偏移(drag方法中的x, y)，可以将其转化为对应网格的坐标。
2. 转化后的坐标如果发生变化，则更新占位节点的坐标。

drag方法中增加的代码如下：

```javascript
// 坐标转换
const nodeX = Math.round(x / opt.cellW);
const nodeY = Math.round(y / opt.cellH);

let currentNode = this.dragrid.currentNode;

// 发生移动
if(currentNode.x !== nodeX || currentNode.y !== nodeY) {
    currentNode.x = nodeX;
    currentNode.y = nodeY;
}
```

## nodes重排及上移

本节核心点有两个：

1. 用一个二维数组来表示网格，这样节点的位置信息就可以在此二维数组中标记出来了。
2. nodes中只要某个节点发生变化，就要重新排版，要将每个节点尽可能地上移。

### 二维数组的构建

```javascript
getArea(nodes) {
  let area = [];
  nodes.forEach(n => {
    for(let row = n.y; row < n.y + n.h; row++){
      let rowArr = area[row];
      if(rowArr === undefined){
        area[row] = new Array();
      }
      for(let col = n.x; col < n.x + n.w; col++){
        area[row][col] = n.id;
      }
    }
  });
  return area;
}
```
按需可以动态扩展该二维数据，如果某行没有任何节点占位，则实际存储的是一个undefined值。否则存储的是节点的id值。

### 布局方法

dragird.vue中watch了nodes，发生变化后会调用layout方法，代码如下：

```javascript
/**
 * 重新布局
 * 只要有一个节点发生变化，就要重新进行排版布局
 */
layout() {
  this.nodes.forEach(n => {
    const y = this.moveup(n);
    if(y < n.y){
      n.y = y;
    }
  });
},

// 向上查找节点可以冒泡到的位置
moveup(node) {
  let area = this.area;
  for(let row = node.y - 1; row > 0; row--){
    // 如果一整行都为空，则直接继续往上找
    if(area[row] === undefined) continue;
    for(let col = node.x; col < node.x + node.w; col++){
      // 改行如果有内容，则直接返回下一行
      if(area[row][col] !== undefined){
        return row + 1;
      }
    }
  }
  return 0;
}
```

布局方法layout中遍历所有节点，moveup方法返回该节点纵向可以上升到的位置坐标，如果比实际坐标小，则进行上移。moveup方法默认从上一行开始找，直到发现二维数组中存放了值（改行已经有元素了），则返回此时行数加1。

到这里，拖拽节点移动时，占位节点会尽可能地上移，如果只有一个节点，那么占位节点一直在最上面移动。

## 相关节点的下移

拖拽节点移动时，与拖拽节点发生碰撞的节点及其下发的节点，都先下移一定距离，这样拖拽节点就可以移到相应位置，最后节点都会发生上一节所说的上移。

请看dragrid.vue中的overlap方法：

```javascript
overlap(node) {
  // 下移节点
  this.nodes.forEach(n => {
    if(node !== n && n.y + n.h > node.y) {
      n.y += node.h;
    }
  });
}
```
`n.y + n.h > node.y` 表示可以与拖拽节点发生碰撞，以及在拖拽节点下方的节点。

在dragdrop.drag中会调用该方法。

注意目前该方法会有问题，没有考虑到如果碰撞节点比较高，则`n.y += node.h`并没有将该节点下沉到拖拽节点下方，从而拖拽节点会叠加上去。后面会介绍解决方法。

## 缩放

上面的思路都理解之后，缩放其实也是一样的，主要还是要进行坐标转换，坐标发生变化后，就会调用overlap方法。

```javascript
resize(event) {
  const opt = this.dragrid.cfg;

  // 之前
  const x1 = this.currentNode.x * opt.cellW + this.offsetX,
      y1 = this.currentNode.y * opt.cellH + this.offsetY;
  // 之后
  const x2 = event.pageX - this.containerOffset.left,
      y2 = event.pageY - this.containerOffset.top;
  // 偏移
  const dx = x2 - x1, dy = y2 - y1;
  // 新的节点宽和高
  const w = this.currentNode.w * opt.cellW + dx,
      h = this.currentNode.h * opt.cellH + dy;

  // 样式设置
  this.dragElement.style.cssText += ';width:' + w + 'px;height:' + h + 'px;';

  // 坐标转换
  const nodeW = Math.round(w / opt.cellW);
  const nodeH = Math.round(h / opt.cellH);

  let currentNode = this.dragrid.currentNode;

  // 发生移动
  if(currentNode.w !== nodeW || currentNode.h !== nodeH) {
      currentNode.w = nodeW;
      currentNode.h = nodeH;
      this.dragrid.overlap(currentNode);
  }
}
```

根据鼠标距拖拽容器的距离的偏移，来修改节点的大小（宽和高），其中x1为鼠标点击后距离容器的距离，x2为移动一段距离之后距离容器的距离，那么差值dx就为鼠标移动的距离，dy同理。

到这里，插件的核心逻辑基本上已经完成了。

## [fix]解决碰撞位置靠上的大块，并没有下移的问题

overlap修改为：

```javascript
overlap(node) {
  let offsetUpY = 0;

  // 碰撞检测，查找一起碰撞节点里面，位置最靠上的那个
  this.nodes.forEach(n => {
    if(node !== n && this.checkHit(node, n)){
      const value = node.y - n.y;
      offsetUpY = value > offsetUpY ? value : offsetUpY;
    }
  });

  // 下移节点
  this.nodes.forEach(n => {
    if(node !== n && n.y + n.h > node.y) {
      n.y += (node.h + offsetUpY);
    }
  });
}
```
offsetUpY 最终存放的是与拖拽节点发生碰撞的所有节点中，位置最靠上的节点与拖拽节点之间的距离。然后再下移过程中会加上该offsetUpY值，确保所有节点下移到拖拽节点下方。

这个插件的核心逻辑就说到这里了，读者可以自己解决如下一些问题：

1. 缩放限制，达到最小宽度就不能再继续缩放了。
2. 拖拽控制滚动条。
3. 拖拽边界的限制。
4. 向下拖拽，达到碰撞节点1/2高度就发生换位。

最后让我们再次感谢一下田老师，呱唧呱唧~~~







