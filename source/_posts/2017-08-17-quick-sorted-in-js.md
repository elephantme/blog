---
title: 使用JavaScript实现快速排序
date: 2017-08-20 18:38:09
tags: algorithm
categories: algorithm
---

## 概念

*   在数据集之中，选择一个元素作为"基准"（pivot），可以随机选择。这里我们选择最后一位。
* 将小于基准的元素移到基准的左边，大于基准的元素移到基准的右边。
* 对基准左边和右边不断重复上面两步，直到所有子集只剩下一个元素为止。

<!-- More -->

## 举例

下面只列举了一轮循环是如何进行分区的。

![JavaScript quick sort](/images/quickSort.png)

* 初始化时已最后一位元素“5”作为基准，同时分区索引（partitionIndex）为0。
* i=0时，3<5，将i=0和partitionIndex=0的元素互换位置（没发生变化），之后partitionIndex++（变为1）。紧接着7和8都大于5，不做任何处理。i=3时，4<5, 将i=3和partitionIndex=1的元素互换位置，同时partitionIdex++（变为2）。
* i=4时，2<5，将i=4和partitionIndex=2的元素互换位置，同时partitionIdex++（变为3）。
* i=5时，1<5，将i=5和partitionIndex=3的元素互换位置，同时partitionIdex++（变为4）。之后就是基准元素，循环结束。
* 循环结束后，我们将基准元素和partitionIndex=4的元素互换位置。
* 经过这样一轮循环后，小于基准的元素都移动到了左边，大于基准的元素都移动到了右边。

## Js的实现

```javascript
// 交换数组中两元素位置
function swap(arr, i, j){
   var temp = arr[i];
   arr[i] = arr[j];
   arr[j] = temp;
}

/**
 * 对数组进行分区
 * @param  {[Array]} arr   [要分区的数组]
 * @param  {[Number]} pivot [基准元素下标索引]
 * @param  {[Number]} left  [左边界]
 * @param  {[type]} right [右边界]
 * @return {[Number]}       [分区结果的索引]
 */
function partition(arr, pivot, left, right){
  var pivotValue = arr[pivot],
      partitionIndex = left;

  for(var i = left; i < right; i++){
    // 如果当前元素小于基准元素，则交换位置，并递增分区索引
    if(arr[i] < pivotValue){
      swap(arr, i, partitionIndex);
      partitionIndex++;
    }
  }
  // 循环结束后，交换基准元素和分区索引对应的元素
  swap(arr, right, partitionIndex);
  return partitionIndex;
}

/**
 * 对数组进行快速排序
 * @param  {[Array]} arr   [要进行排序的数组]
 * @param  {[Number]} left  [要进行排序的起始位置]
 * @param  {[Number]} right [要进行排序的结束位置]
 * @return {[Array]}       [排好序的数组，仍然为原数组]
 */
function quickSort(arr, left, right){
  var len = arr.length, 
   pivot,  // 基准元素下标
   partitionIndex; // 分区索引

  left = typeof left === 'undefined' ? 0 : left;
  right = typeof right === 'undefined' ? len - 1 : right;

  if(left < right){
    // 以最后一个元素为基准
    pivot = right; 
    // 进行分区，并得到分区索引，该索引将数组分为两部分，再递归进行调用排序 
    partitionIndex = partition(arr, pivot, left, right);
    
    // 对左右两分区分别递归排序
    quickSort(arr, left, partitionIndex - 1);
    quickSort(arr, partitionIndex + 1, right);
  }
  return arr;
}
```

