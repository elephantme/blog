---
layout: post
title:  "svg坐标系统及基本图形介绍"
date:   2016-12-7
categories: svg
---

## 1. 坐标系统

### 1.1 viewPort

1. 原点在左上角，x轴向右递增，y轴向下递增
2. 通过width、height来确定视口大小，默认单位为px

<!--more-->

3. 支持的单位
   1. em 默认字体的大小，通常相当于文本行高
   2. ex 字母x的高度
   3. px 像素 
   4. pt 点（1/72 英寸）
   5. pc 12点（1/6 英寸）
   6. cm 厘米
   7. mm 毫米
   8. in 英寸
4. 视口单位与所绘制图形单位不同也可以

### 1.2 viewBox

1. 用户自定义的新的坐标系统
2. viewBox属性有四个参数，分别表示叠加在视口上的用户坐标系统的最小x坐标、最小y坐标、宽度和高度

### 1.3 宽高比

> viewBox的宽高比与viewPort的宽高比不一致时，svg有三种处理方式：
> 1. 按较小尺寸等比例缩放图形，以使图形完全填充视口。
> 2. 按较大尺寸等比例缩放图形并裁剪掉超出视口的部分。
> 3. 拉伸和挤压绘图以使其恰好填充新的视口(完全不保留宽高比)。
>
> 第一种情况会出现某一个方向要比视口小，需要解决放置在哪里，就是对齐的问题。
> 第二种情况会出现某一个方向要比视口大，需要解决哪个区域被裁剪掉，如左侧、右侧还是两侧。

**preserveAspectRatio指定对齐方式**

> preserveAspectRatio="alignment [meet | slice]" 
> 其中alignment指定轴位置，由xMin、xMid、xMax与yMin、yMid、yMax组合而成，默认为xMidYMid meet。
> meet为第一种情况，slice为第二种情况，要进行裁剪。
> preserveAspectRatio="none" 为第三种情况。

### 1.4 嵌套坐标系统

> 可以在任何时候将另一个svg元素插入到文档中来建立新的视口和坐标系统。其效果相当于一个迷你画布。

## 2. 基本形状

### 2.1 线段

1. `<line x1="start-x" y1="start-y" x2="end-x" y2="end-y" />`
2. stroke-width 笔画宽度
3. stroke: 笔画颜色
4. stroke-opacity 不透明度
5. stroke-dasharrray 一系列数字，表示线的长度和空隙的长度，数字之间用逗号或者空格分隔

### 2.2 矩形

1. `<rect x="左上角x坐标" y="左上角y坐标" width="宽度" height="高度" />`
2. fill 填充颜色，默认为黑色
3. fill-opacity 不透明度
4. 支持线段stroke相关属性
5. rx和ry指定x轴或者y轴圆角的半径，如果只设置一个则认为它们相等

### 2.3 圆和椭圆

1. `<circle cx="圆心横坐标" cy="圆心纵坐标" r="半径" />`
2. `<ellipse cx="圆心横坐标" cy="圆心纵坐标" rx="x方向半径" ry="y方向半径" />`

### 2.4 多边形

1. `<polygon points="一系列的x/y坐标对" />`
2. 填充边界交叉的多边形，有两种算法
  1. 非零缠绕原则(顺时针值加1，逆时针减1，如果结果为0，则认为该点在图形外部，否则为内部)
  2. 奇偶数原则 (总数为奇数为内部，偶数为外部)

### 2.5 折线

1. `<polyline points="一系列的x/y坐标对" />` 不会闭合

### 2.6 线帽和连接线

1. stroke-linecap butt 精确地与起止位置对齐 round square 都超过了真实位置
2. stroke-linejoin miter(尖的) round(圆的) bevel(平的)

