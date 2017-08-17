---
title: 使用JavaScript实现选择排序
date: 2017-08-17 17:00:03
tags: algorithm
categories: algorithm
---

## 概念

在选择排序中，我们先假设数组中第一个元素最小，然后从下个元素开始寻找比该元素小的其它元素，如果找到，则在一次循环结束后交换这两个元素的位置。这之后位于数组中第一个位置中的元素就是最小的了，也就是1位置前的元素是有序的了。

接着我们假设第二个元素在该数组剩余元素中是最小的，依次遍历其它元素从中找出最小元素，并在循环结束后交换位置。这之后第一个和第二个元素已经排好顺序了。按照同样的方式来处理其它剩余元素，之后就得到一个有序的数组。

<!-- More -->

下面看一个例子

### Step 1: 初始化数组
![JavaScript select sort](/images/select-sorted-1.png)

### Step 2:

先假设第一个元素最小，将它和第二个元素进行比较。

![JavaScript select sort](/images/select-sorted-2.png)

因为2<3，所以2仍然是最小元素。

### Step 3:
![JavaScript select sort](/images/select-sorted-3.png)

将2和5进行比较，结果2仍然是最小元素。

### Step 4:
![JavaScript select sort](/images/select-sorted-4.png)

这次，2<1，所以最小元素会变为1，我们会记录其下标索引位置。

![JavaScript select sort](/images/select-sorted-5.png)

现在我们有最新的最小元素，我们会用它和余下的元素进行比较，目的是要找到最小的元素。

### Step 5:
![JavaScript select sort](/images/select-sorted-6.png)

因为1<4，所以在这次循环中1是最小的元素，接下来像下面那样进行换位：

![JavaScript select sort](/images/select-sorted-7.png)

经过这次循环，我们找到了第一个位置的最小元素。接着会找余下元素的最小元素，将它和第二个位置上的元素进行交换。这次我们假设第二个元素是最小元素，重复上面的1到5步。经过第二轮循环，数组如下：

### 第二轮
![JavaScript select sort](/images/select-sorted-8.png)

### 第三轮
![JavaScript select sort](/images/select-sorted-11.png)

### 第四轮
![JavaScript select sort](/images/select-sorted-9.png)

### 第五轮
![JavaScript select sort](/images/select-sorted-10.png)

## JavaScript的实现

```javascript
function selectSort(arr){
  var i,j,tmp,minIndex,len=arr.length;
  for(i=0; i<len; i++){
    // 每次循环暂定首个元素为最小
    minIndex = i;
    for(j=i+1; j<len; j++){
      if(arr[j] < arr[minIndex]){
        // 更新最小下标
        minIndex = j;
      }
    }
    // 循环结束后如果最小元素的下标和首个元素的不一致，则换位
    if(i != minIndex){
      tmp = arr[i];
      arr[i] = arr[minIndex];
      arr[minIndex] = tmp;
    }
  }
  return arr;
}
```

原文链接 [Sorting Algorithms: Selection Sort using JavaScript](http://codingmiles.com/sorting-algorithms-selection-sort-using-javascript/)