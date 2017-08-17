---
title: 使用JavaScript实现归并排序
date: 2017-08-17 18:38:00
tags: algorithm
categories: algorithm
---

在计算机科学领域里，归并排序是一种非常有用的排序算法。它的时间复杂度为O(n log n)，使得它非常高效。另外它也是一种稳定排序（类似插入排序）。正是因为这些优点，火狐和Safari浏览器中`Array.prototype.sort()`底层排序算法就使用的归并排序。

<!-- More -->

归并排序的算法思路是，将两个有序的数组合并为一个有序数组往往比直接处理一个无序数组容易。为此，归并排序要创建n个只有一个元素的列表，其中n为要排序的列表的长度，然后再将这些列表合并为单个有序列表。

将两个有序列表合并为一个有序列表的算法比较容易。假设有两个列表，A和B。分别取出第一个元素进行比较，将较小元素放到一个新的结果数组中，同时从列表中删除该元素，执行这样的操作直到某个列表已没有任何元素。然后将结果数组，A和B连接起来就得到了一个有序的数组。

```javascript
function merge(left, right){
  var result = [];

  while(left.length > 0 && right.length > 0){
    if(left[0] < right[0]){
      result.push(left.shift());
    }else{
      result.push(right.shift());
    }
  }

  return result.concat(left, right);
}
```

该方法目的是合并两个数组left和right，result用来存放从left或者right取出来的元素。因为每次都是从left和right中取较小的元素，而它们本身又是有序的，所以result中存放的元素也就有序了。

`merge()`方法比较简单，但是你需要构造两个有序列表。就像上面说的，可以通过将数组分割为一个个只有一个元素的数组，然后再进行合并。这可以通过递归的方式来实现：

```javascript
function mergeSort(arr){
  if(arr.length <=1) return arr;
  var middle = Math.floor(arr.length / 2);
  var left = arr.slice(0, middle);
  var right = arr.slice(middle);
  return merge(mergeSort(left), mergeSort(right));
}
```

第一个需要注意的是当数组元素个数为0或者1时，不需要进行排序，直接返回就可以。而多个元素的数组被分割为两部分：left和right。先递归处理left，直到它被分割为一个个只有一个元素的数组，然后开始向上逐级调用`merge`来合并为一个有序的数组。之后用类似的方式处理right，也得到一个有序的数组，最后再调用一次`merge`方法，得到最终的有序数组。

在实际开发中，我们优先调用原生的`Array.prototype.sort()`方法来实现排序。浏览器底层会提供高效的排序实现，需要注意的是，并不是所有的实现都采用稳定排序，如果你对这个有要求，就需要自己来实现排序了。

原文链接 [Computer science in JavaScript: Merge sort](https://www.nczonline.net/blog/2012/10/02/computer-science-and-javascript-merge-sort/)
