---
title: 使用JavaScript实现插入排序
date: 2017-08-17 14:25:43
tags: 
categories: 
---

## 概念

在插入排序中，我们将初始无序的数组分为两部分，有序部分和无序部分。有序部分初始化时只有一个元素（一般为数组第一个元素）。之后我们将无序部分中的元素一个个地插入到有序部分，每次插入的时候会寻找合适的位置并扩展有序部分。

<!-- More -->

下面一起看一个示例：

### Step 1: 初始化数组
![JavaScript inserted sort](/images/is-step1.png)

### Step 2:
![JavaScript inserted sort](/images/is-step2.png)

元素5在有序部分，第一次我们从无序部分挑选出元素3，并将它插入到正确的位置上。我们先创建元素3的副本，然后将它和有序部分中最后的元素进行比较，也就是5。

### Step 3:
![JavaScript inserted sort](/images/is-step3.png)

因为5>3，我们就将无序部分的3用5来替换，同是变为有序部分。之后，我们将副本3插入到之前5的位置上。此时有序部分已经扩展为2个元素，无序部分减少为3个元素了。

### Step 4:
![JavaScript inserted sort](/images/is-step4.png)

接下来从无序部分选择下一个元素1。再次创建这个元素的一个副本，将它和有序部分依次进行比较。

### Step 5:
![JavaScript inserted sort](/images/is-step5.png)

因为5>1，我们将元素1用5代替并扩展有序部分。紧接着副本1和3进行比较。

### Step 6:
![JavaScript inserted sort](/images/is-step6.png)

因为3>1，我们需要将有序部分中的3移动到下一个位置，因此会将第二个位置中的5使用3来代替。之后没有其它元素可以进行比较，我们就将1插入到第一个位置。

### Step 7:
![JavaScript inserted sort](/images/is-step7.png)

下一个元素2按照同样的处理方式来处理3和5。

![JavaScript inserted sort](/images/is-step8.png)

![JavaScript inserted sort](/images/is-step9.png)

### Step 8:
![JavaScript inserted sort](/images/is-step10.png)

之后，副本2和1进行比较，因为1<2，需要将2插入到1之后，我们就将第二个位置上的3用2来替换。

![JavaScript inserted sort](/images/is-step11.png)

### Step 9:
![JavaScript inserted sort](/images/is-step12.png)

同样的方式来处理元素4。

## 时间复杂度

|最好|平均|最坏|
|---------|--------|--------|
| O(n) | O(n^2) | O(n^2) |

## JavaScript的实现

```javascript
function insertSort(arr){
  var tmp, i, j;
  // 从第二个元素开始比较
  for(i=1; i<arr.length; i++){
    // 元素的副本
    tmp = arr[i];
    // 和前面有序部分进行比较，如果元素副本小于有序部分中的元素，则移动该元素
    for(j=i-1; j>=0 && arr[j] > tmp; j--){
      // 移动
      arr[j+1] = arr[j];
    }
    // 将副本元素插入到有序部分合适的位置上
    arr[j+1] = tmp;
  }
  return arr;
}
```

原文链接 [Sorting Algorithms: Insertion Sort using JavaScript](http://codingmiles.com/sorting-algorithms-insertion-sort-using-javascript/)
