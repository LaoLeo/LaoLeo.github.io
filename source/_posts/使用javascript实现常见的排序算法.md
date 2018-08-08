---
title: 使用javascript实现常见的排序算法
date: 
tags: [js, 算法]
categories: [js]
---

![我是图片](https://pic3.zhimg.com/80/v2-d3a17aea09ae61c6a4d3e982415b5268_hd.jpg)

# 前言
算法，一门老生常谈的学科，最近在复习算法的知识，随作为切图的我，在平常工作中很难用的上这些知识，但是加强自身功底还是很有必要的。写下笔记记录成长的点滴，往后回头也还可以复习复习。在学习之余也乐于分享，好在在进击的路上，与诸君共勉。

<!--more-->

# 正文
## 评述算法优劣的术语
**稳定：** 如果在序列里面本来a位于b前面，当a === b时，不交换a和b的位置；  **不稳定：** 就是在前者的基础上，a和b的位置交换了

**内排序：** 所有的操作都在内存中完成； **外排序：** 由于数据量太大，把数据存放在磁盘上，而排序需要通过磁盘和内存的数据传输才能实现

**时间复杂度：** 一个算法运行完所需要时间的大小；**空间复杂度：** 一个算法运行需要的内存大小。

## 排序算法
**排序算法的对比**

![image](https://user-gold-cdn.xitu.io/2016/11/29/4abde1748817d7f35f2bf8b6a058aa40?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

**图中名词解析:**  n: 数据规模; k:“桶”的个数;In-place: 占用常数内存，不占用额外内存; Out-place: 占用额外内存

对了算法术语有了初步的了解后，我们来看看排序算法的实现

### 1.冒泡算法
**思路:**  
(1)从前向后遍历元素，对比相邻的两个元素，根据条件交换两个元素  
(2)重复的对比相邻的两个数，把小的值移到前面，此过程就像冒泡一样

**javascript代码实现**
```javascript
// 构造排序算法类骨架
function ArraySort() {
    this.bubbleSort = function() {}
    this.selectSort = function() {}
    this.insertSort = function() {}
    this.mergeSort = function() {}
    this.quickSort = function() {}
    this.heapSort = function() {}
    
    // 交换
    function swap(array, i, j) {
        [array[i], array[j]] = [array[j], array[i]]
    }
}

// 实现
this.bubbleSort = function(array) {
    let length = array.length

    for(let i = 0; i < length; i++) {
        for(let j = 0; j < length-1-i; j++) {
            if(array[j] > array[j+1]) {
                swap(array, j, j+1)
            }
        }
    }
}
```

### 2.选择排序
**思路:**  
(1)在一层遍历的时候，选择首元素认为是最小值
(2)在二层遍历中，将每个元素跟这个伪最小值对比，选择出比其更小的元素
(3)二层遍历结束前，将这两个元素交换
(4)重复123过程

**javascript代码实现**
```javascript
this.selectSort = function(array) {
    let length = array.length,minIndex

    for(let i = 0; i < length-1; i++) {
        minIndex = i
        for(let j = i+1; j < length; j++) {
            if(array[minIndex] > array[j]) {
                minIndex = j
            }
    }
    if(minIndex != i) swap(array, i, minIndex)
}
```

### 3.插入排序
**思路：**  
(1)类似拿到扑克牌后，将小牌插到前面那样。从下标1开始（递增），从后往前遍历
(2)抽出元素，前一个元素与其对比，大的就往后插
(3)元素往前移动，重复2，当元素不能往前移动了，就把抽出的元素插到空的的位置上
(4)重复23

**javascript代码实现**
```javascript
this.insertSort = function(array) {
    let length = array.length, temp, j

    for(let i = 1; i < length; i++) {
        j = i
        temp = array[i]

        while(j>0 && array[j-1] > temp) {
            array[j] = array[j-1]
            j--
        }

        array[j] = temp
    }
}
```

### 4.归并排序
> 前面三个排序的时间复杂度都是O(n^2)，归并排序采用了分治的思想，将复杂的问题分割成小单元，小单元在问题解决后再合并成解，利用了递归，算法复杂度为O(nlogn).

**思路：**  
(1)将数组对折分半  
(2)递归对折到最小单元，比较两个数的大小，交换

**javascript代码实现**
```javascript
this.mergeSort = function(array) {
    mergeRec(array)
    
    function mergeRec(array) {
        if (array.length === 1) return array

        let left, right, imid
        imid = Math.floor(array.length/2)
        left = array.slice(0, imid)
        right = array.slice(imid)
    
        return merge(mergeRec(left), mergeRec(right))
    }
    
    function merge(left, right) {
        let result = [], il = 0, ir = 0
        while (il < left.length && ir < right.length) {
            if (left[il] < right[ir]) {
                result.push(left[il++])
            } else {
                result.push(right[ir++])
            }
        }
    
        while (il < left.length) {
            result.push(left[il++])
        }
    
        while (ir < right.length) {
            result.push(right[ir++])
        }
        
        return result
    }
}
```

### 5.快速排序
> 快排是使用最广泛的排序算法，跟归并一样也使用了分治思想，时间复杂度也为O(nlogn)

**思路：**  
(1)取数组中间的值作为参考值，从首尾向中间遍历，从中间值左边找到比其大的数，再从右边找到比其小的数，交换两者，直到左边游标大于右边的游标
(2)把下标0到左边游标-1作为一个区间，把左边游标到length-1作为一个区间
(3)递归12过程

**javascript代码实现**
```javascript
this.quickSort = function() {
    quick(array, 0, array.length-1)
    
    function quick(array, left, right) {
        if (array.length < 2) return

        let index = partition(array, left, right)
    
        if (left < index - 1) {
            quick(array, left, index - 1)
        } 
        if (index < right){
            quick(array, index, right)
        }
    }
    
    function partition(array, left, right) {
        let pivot = array[Math.floor((left + right) / 2)]
    
        while (array[left] < pivot) {
            left++
        }
    
        while (array[right] > pivot) {
            right--
        }
    
        if (left <= right) {
            swap(array, left, right)
        }
    
        return ++left
    }
}
```

### 6.堆排序
> 堆排序把数组当作二叉树来管理，也属于属于高效的排序算法之一。堆结构符合一下原则：根节点为堆中最大的值，父节点总是大于子节点值。

**思路：**  
(1)构造一个满足array[parent(i)] >= array[i]的二叉树
(2)交换堆中的第一个元素和最后一个元素，此时最大值已经排好位置，缩小堆大小。
(3)此时堆可能不符合原则了，需要转化堆，也就是将新堆中的最大值放置根节点。
(4)重复23

**javascript代码实现**
```javascript
this.heapSort = function() {
    let heapsize = array.length
    if (heapsize < 2) return

    // 构造堆，父节点总是大于子节点
    for(let i = Math.floor(heapsize / 2); i >= 0; i--) {
        heapify(array, heapsize, i)
    }

    while (heapsize > 0) {
        heapsize--
        swap(array, 0, heapsize)
        heapify(array, heapsize, 0)
    }
    
    function heapify(array, heapsize, i) {
        let left, right, parenti
        left = i*2+1
        right = i*2+2
        largest = i
    
        if (left < heapsize && array[left] > array[largest]) {
            largest = left
        }
        if (right < heapsize && array[right] > array[largest]) {
            largest = right
        }
        
        if (largest !== i) {
            swap(array, largest, i)
            heapify(array, heapsize, largest)
        } 
    }
}
```

看到这你可能会想，“兄dei Array原型上不是有sort()吗，何必自己写排序算法呀”。的确，在开发过程中我们很少情况下写排序算法，大部分情况前端拿到的数据是后端已经排后序的了，有时连sort也不用掉用，因为前端排序数据很花性能，我们还是花多点时间在用户体验上会比较划算。

写这个常见的排序算法完全是自己加固技术功底的玩意，但是有趣的问题来了，sort的实现原理是啥呀？要不我们来实现一个。

## Array.prototype.sort的实现
在MDN上sort的定义  
**语法**
```javascript
arr.sort([compareFunction])
```
**参数**  
compareFunction 可选  
用来指定按某种顺序进行排列的函数。如果省略，元素按照转换为的字符串的各个字符的Unicode位点进行排序。

**返回值**
排序后的数组。请注意，数组已原地排序，并且不进行复制。


**javascript代码实现**
```javascript
this.arraySortFn(array, cFn) {
    let _cFn
    if (cFn === undefined) {
        _cFn = _compareStringUnicode
    }else if (typeof cFn === 'function') {
        _cFn = cFn
    }else {
        throw new Error('compareFn must be a function')
    }

    _selectSort(array, _cFn)
    
    // 改写选择排序，传入判断条件
    function _selectSort(array, cFn) {
        let l = array.length, selectIndex
        for (let i = 0; i < l - 1; i++) {
            // 假设的最值下标
            selectIndex = i
            for (let j = i + 1; j < l; j++) {
                if (cFn(array[selectIndex], array[j]) > 0) selectIndex = j
            }
            if (selectIndex !== i) swap(array, i, selectIndex)
        }
    }
    
    // 默认比较字符串的Unicode编码
    function _compareStringUnicode(a, b) {
        if(
            (typeof a !== 'string' && typeof a !== 'number' )||
            (typeof b !== 'string' && typeof b !== 'number')
        ) {
            throw new Error(`${a} / ${b} must be number or string`)
        }

        a = String(a)
        b = String(b)
        let l = a.length > b.length ? b.length : a.length

        for(let i=0; i<l; i++) {
            if(a.charCodeAt(i) === b.charCodeAt(i)) continue

            return a.charCodeAt(i) > b.charCodeAt(i)
        }

        return a.length > b.length ? true : false
    }
}
```

参考资料：  
1.[十大经典排序算法总结（JavaScript描述）](https://juejin.im/post/57dcd394a22b9d00610c5ec8)  
2.《学习javascript数据结构与算法》
