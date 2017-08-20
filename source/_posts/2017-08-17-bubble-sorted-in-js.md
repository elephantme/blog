---
title: 使用JavaScript实现冒泡排序
date: 2017-08-17 18:37:48
tags: algorithm
categories: algorithm
---

## 概念

在冒泡排序中，是从数组中第一个元素开始并且依次相邻元素之间进行比较。如果第一个元素大于第二个元素，则交换它们的位置。否则第二个与第三个元素进行比较，以此类推。在第一轮循环结束后，会将最大的元素放到数组的末尾。
<!-- More -->

下面看例子：

### Step 1:初始化数组
![JavaScript select sort](/images/BubbleSort_initial.png)

### Step 2:第一个与第二个比较
![JavaScript select sort](/images/BubbleSort_step1.png)

4>3，则交换它们的位置。

### Step 3:第二个与第三个比较
![JavaScript select sort](/images/BubbleSort_step2.png)

4<5, 则不发生交换。

### Step 4:第三个与第四个比较
![JavaScript select sort](/images/BubbleSort_step3.png)

5>1, 则交换它们的位置。

### Step 5:第四个与第五个比较
![JavaScript select sort](/images/BubbleSort_step4.png)

5<7，则不发生交换。

### Step 6:第五个与第六个比较
![JavaScript select sort](/images/BubbleSort_step5.png)

7>6, 则交换它们的位置。

### Step 7:第六个与第七个比较
![JavaScript select sort](/images/BubbleSort_step6.png)

7<8, 则不发生交换。

### Step 8:第七个与第八个比较
![JavaScript select sort](/images/BubbleSort_step7.png)

8>2, 则交换它们的位置。

经过第一轮循环，将最大元素8放置到数组末尾，如下：

![JavaScript select sort](/images/BubbleSort_pass1.png)

第一轮循环结束后，下一次循环还是从数组开始位置进行比较，直到倒数第二个位置。倒数第一已经排好序了，就不需要再进行比较了。

### 第二轮
![JavaScript select sort](/images/BubbleSort_pass2.png)

### 第三轮
![JavaScript select sort](/images/BubbleSort_pass3.png)

### 第四轮
![JavaScript select sort](/images/BubbleSort_pass4.png)

### 第五轮
![JavaScript select sort](/images/BubbleSort_pass5.png)

等等...

### 最后
![JavaScript select sort](/images/BubbleSort_pass7.png)

## 时间复杂度

假设数组长度为N，第一轮循环需要进行N次比较，第二轮需要进行N-1次比较，第三轮为N-2次，以此类推。

总的比较次数应该为：

N + (N-1) + (N-2) + …     ≈  (N * (N-1)) / 2 ， 如果 N → ∞ ，则结果 ≈ N²。

**因此冒泡排序的时间复杂度为：O(N²)**

## JavaScript的实现

```javascript
function bubbleSort(arr){
  var i,j,tmp,len=arr.length;
  for(i=0; i<len; i++){
    for(j=0; j<len-i; j++){
      // 相邻元素进行比较
      if(arr[j] > arr[j+1]){
        tmp = arr[j];
        arr[j] = arr[j+1];
        arr[j+1] = tmp;
      }
    }
  }
  return arr;
}
```

原文链接 [Sorting Algorithms: Bubble Sort using javascript](http://codingmiles.com/sorting-algorithms-bubble-sort-using-javascript/)