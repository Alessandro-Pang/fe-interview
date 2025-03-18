---
weight: 3005000
date: '2025-03-18T02:00:27.626Z'
draft: false
author: Kismet
title: 求数组中最大的两个值
icon: javascript
toc: true
description: 求数组中最大的两个值
tags:
  - JavaScript
  - 算法
  - 数组
---

## 考察点分析

这道题主要考察以下几个方面：

1. 数组遍历和比较操作
2. 时间复杂度优化
3. 边界情况处理能力
4. 算法思维

## 技术解析

### 原理剖析

求数组中最大的两个值，有多种解决方案：

1. **排序法**：将数组排序后取最后两个元素
2. **两次遍历法**：第一次找最大值，第二次找第二大值
3. **单次遍历法**：维护两个变量，分别记录最大值和第二大值

其中，单次遍历法的时间复杂度最优，为O(n)。

### 常见误区

1. **忽略数组长度检查**：当数组长度小于2时，无法返回两个最大值
2. **重复计算最大值**：在两次遍历法中，第二次遍历时没有排除第一次找到的最大值
3. **未考虑相等元素**：当有多个元素值相同且为最大值时的处理
4. **未考虑负数情况**：初始化第二大值为0可能导致负数数组出错

---

## 问题解答

### 1. 排序法

  ```javascript
  function findTwoLargest(arr) {
    if (!Array.isArray(arr) || arr.length < 2) {
      return "数组长度不足";
    }

    // 对数组进行降序排序
    const sortedArr = [...arr].sort((a, b) => b - a);

    // 返回最大的两个值
    return [sortedArr[0], sortedArr[1]];
  }
  ```

  **时间复杂度：O(n log n)，主要受排序算法影响**

### 2. 两次遍历法

  ```javascript
    function findTwoLargest(arr) {
      if (!Array.isArray(arr) || arr.length < 2) {
        return "数组长度不足";
      }

      // 第一次遍历找最大值
      let max = -Infinity;
      let maxIndex = -1;

      for (let i = 0; i < arr.length; i++) {
        if (arr[i] > max) {
          max = arr[i];
          maxIndex = i;
        }
      }

      // 第二次遍历找第二大值
      let secondMax = -Infinity;

      for (let i = 0; i < arr.length; i++) {
        if (i !== maxIndex && arr[i] > secondMax) {
          secondMax = arr[i];
        }
      }

      return [max, secondMax];
    }
  ```

  **时间复杂度：O(n)，只需要遍历数组两次**

### 3. 单次遍历法

  ```javascript
  function findTwoLargest(arr) {
    if (!Array.isArray(arr) || arr.length < 2) {
      return "数组长度不足";
    }

    let max = -Infinity;
    let secondMax = -Infinity;

    for (let i = 0; i < arr.length; i++) {
      if (arr[i] > max) {
        // 当前值大于最大值，更新最大值和第二大值
        secondMax = max;
        max = arr[i];
      } else if (arr[i] > secondMax) {
        // 当前值大于第二大值但小于最大值，更新第二大值
        secondMax = arr[i];
      }
    }

    return [max, secondMax];
  }
  ```

  **时间复杂度：O(n)，只需要遍历数组一次**

### 4. 测试用例

  ```javascript
  // 测试
  console.log(findTwoLargest([1, 2, 3, 4, 5])); // [5, 4]
  console.log(findTwoLargest([5, 5, 4, 3, 2])); // [5, 5]
  console.log(findTwoLargest([-1, -2, -3, -4])); // [-1, -2]
  console.log(findTwoLargest([10])); // "数组长度不足"
  ```

---

## 深度追问

### 1. 如何处理有多个相同的最大值的情况？

如果我们严格定义"两个最大的值"为不同的值，当有多个相同的最大值时，我们需要找出最大值和严格小于最大值的第二大值。

```javascript
function findTwoDistinctLargest(arr) {
  if (!Array.isArray(arr) || arr.length < 2) {
    return "数组长度不足";
  }

  // 先排序去重
  const uniqueSorted = [...new Set(arr)].sort((a, b) => b - a);

  // 如果去重后长度不足2，说明数组中所有元素都相同
  if (uniqueSorted.length < 2) {
    return "没有两个不同的值";
  }

  // 返回最大的两个不同值
  return [uniqueSorted[0], uniqueSorted[1]];
}

// 测试
console.log(findTwoDistinctLargest([5, 5, 5, 3, 2])); // [5, 3]
console.log(findTwoDistinctLargest([5, 5, 5, 5])); // "没有两个不同的值"
```

另一种方法是在单次遍历中处理：

```javascript
function findTwoDistinctLargest(arr) {
  if (!Array.isArray(arr) || arr.length < 2) {
    return "数组长度不足";
  }

  let max = -Infinity;
  let secondMax = -Infinity;

  for (let i = 0; i < arr.length; i++) {
    if (arr[i] > max) {
      secondMax = max;
      max = arr[i];
    } else if (arr[i] < max && arr[i] > secondMax) {
      // 只有当元素严格小于max且大于secondMax时才更新secondMax
      secondMax = arr[i];
    }
  }

  // 检查是否找到了第二大的不同值
  if (secondMax === -Infinity) {
    return "没有两个不同的值";
  }

  return [max, secondMax];
}
```

### 2. 如何优化空间复杂度？

1. 排序法 ：空间复杂度为O(n)，因为创建了新数组

   ```javascript
   // 优化空间复杂度的排序法 - 原地排序
    function findTwoLargestOptimized(arr) {
      if (!Array.isArray(arr) || arr.length < 2) {
        return "数组长度不足";
      }

      // 直接对原数组排序，不创建新数组
      arr.sort((a, b) => b - a);

      return [arr[0], arr[1]];
    }
   ```

   注意：这种方法会修改原数组，如果不希望修改原数组，就无法避免O(n)的空间复杂度。

2. 两次遍历法和单次遍历法 ：空间复杂度均为O(1)，只使用了常数个变量

    ```javascript
    // 单次遍历法的空间复杂度已经是最优O(1)
    function findTwoLargest(arr) {
    if (!Array.isArray(arr) || arr.length < 2) {
      return "数组长度不足";
    }

    let max = arr[0];
    let secondMax = arr[1];

    // 确保max是最大的，secondMax是第二大的
    if (secondMax > max) {
      [max, secondMax] = [secondMax, max];
    }

    // 从第三个元素开始遍历
    for (let i = 2; i < arr.length; i++) {
      if (arr[i] > max) {
        secondMax = max;
        max = arr[i];
      } else if (arr[i] > secondMax) {
        secondMax = arr[i];
      }
    }

    return [max, secondMax];
    }
    ```

    总结：对于这个问题，单次遍历法已经达到了O(1)的最优空间复杂度，无需进一步优化。如果不希望修改原数组，排序法的O(n)空间复杂度是无法避免的。

### 3. 如何处理大数据量的情况？

   对于非常大的数组，可以考虑分治法或并行处理：

  ```javascript
  function findTwoLargestParallel(arr) {
    if (!Array.isArray(arr) || arr.length < 2) {
      return "数组长度不足";
    }

    // 将数组分成多个块
    const chunkSize = 10000; // 根据实际情况调整
    const chunks = [];

    for (let i = 0; i < arr.length; i += chunkSize) {
      chunks.push(arr.slice(i, i + chunkSize));
    }

    // 对每个块找出最大的两个值
    const results = chunks.map(chunk => {
      let max = -Infinity;
      let secondMax = -Infinity;

      for (let i = 0; i < chunk.length; i++) {
        if (chunk[i] > max) {
          secondMax = max;
          max = chunk[i];
        } else if (chunk[i] > secondMax) {
          secondMax = chunk[i];
        }
      }

      return [max, secondMax];
    });

    // 合并结果
    let finalMax = -Infinity;
    let finalSecondMax = -Infinity;

    for (let i = 0; i < results.length; i++) {
      const [max, secondMax] = results[i];

      if (max > finalMax) {
        finalSecondMax = Math.max(finalMax, secondMax);
        finalMax = max;
      } else if (max > finalSecondMax) {
        finalSecondMax = max;
      } else if (secondMax > finalSecondMax) {
        finalSecondMax = secondMax;
      }
    }

    return [finalMax, finalSecondMax];
  }

  // 在实际环境中，可以使用Web Workers或Node.js的Worker Threads实现真正的并行处理
  ```

  在JavaScript中，可以使用Web Workers或Node.js的Worker Threads来实现真正的并行计算，进一步提高大数据量下的处理效率。

### 4. 如何扩展到求前K大的元素？

   求数组中前K大的元素，可以使用最小堆（优先队列）实现：

  ```javascript
  class MinHeap {
    constructor() {
      this.heap = [];
    }

    getParentIndex(i) {
      return Math.floor((i - 1) / 2);
    }

    getLeftChildIndex(i) {
      return 2 * i + 1;
    }

    getRightChildIndex(i) {
      return 2 * i + 2;
    }

    swap(i1, i2) {
      [this.heap[i1], this.heap[i2]] = [this.heap[i2], this.heap[i1]];
    }

    peek() {
      return this.heap[0];
    }

    size() {
      return this.heap.length;
    }

    insert(value) {
      this.heap.push(value);
      this.siftUp(this.heap.length - 1);
    }

    siftUp(index) {
      let parent = this.getParentIndex(index);
      while (index > 0 && this.heap[parent] > this.heap[index]) {
        this.swap(parent, index);
        index = parent;
        parent = this.getParentIndex(index);
      }
    }

    extractMin() {
      if (this.heap.length === 0) return null;
      const min = this.heap[0];
      const last = this.heap.pop();
      if (this.heap.length > 0) {
        this.heap[0] = last;
        this.siftDown(0);
      }
      return min;
    }

    siftDown(index) {
      let minIndex = index;
      const left = this.getLeftChildIndex(index);
      const right = this.getRightChildIndex(index);
      const size = this.heap.length;

      if (left < size && this.heap[left] < this.heap[minIndex]) {
        minIndex = left;
      }

      if (right < size && this.heap[right] < this.heap[minIndex]) {
        minIndex = right;
      }

      if (index !== minIndex) {
        this.swap(index, minIndex);
        this.siftDown(minIndex);
      }
    }
  }

  function findKLargest(arr, k) {
    if (!Array.isArray(arr) || arr.length < k || k <= 0) {
      return "参数不合法";
    }

    // 创建一个最小堆
    const minHeap = new MinHeap();

    // 先将前k个元素加入堆
    for (let i = 0; i < k; i++) {
      minHeap.insert(arr[i]);
    }

    // 遍历剩余元素
    for (let i = k; i < arr.length; i++) {
      // 如果当前元素大于堆顶，则替换堆顶并重新调整堆
      if (arr[i] > minHeap.peek()) {
        minHeap.extractMin();
        minHeap.insert(arr[i]);
      }
    }

    // 从堆中提取所有元素（从小到大）
    const result = [];
    while (minHeap.size() > 0) {
      result.unshift(minHeap.extractMin());
    }

    return result;
  }

  // 测试
  console.log(findKLargest([3, 1, 5, 7, 2, 4, 9, 6], 3)); // [7, 9, 6]
  ```

  时间复杂度：O(n log k)，其中n是数组长度，k是要找的最大元素个数。空间复杂度：O(k)，用于存储堆。
  这种方法对于大数据量且k远小于n的情况特别高效。
