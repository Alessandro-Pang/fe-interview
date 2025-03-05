---
weight: 2700
date: '2025-03-04T06:58:24.482Z'
draft: false
author: zi.Yang
title: 数组操作副作用对比
icon: javascript
toc: true
description: 请说明splice()和slice()方法是否修改原数组，并给出三种不同删除数组最后一个元素的方法实现。
tags:
  - javascript
  - 数组操作
---

## 考察点分析

该题主要考察以下核心能力维度：

1. **数组方法副作用理解**：准确辨别修改原数组与返回新数组的方法差异
2. **数组操作熟练度**：掌握多种常用数组操作方法的具体应用场景
3. **基础API原理认知**：理解length属性与数组内存分配的关系

具体技术评估点：

- splice()与slice()的副作用差异
- pop()、splice()、length属性修改三种删除方式的实现原理
- 负索引在splice()中的特殊处理

## 技术解析

### 关键知识点

splice()方法 > slice()方法 > 数组length属性

### 原理剖析

- **splice()**：通过指定起始位置和删除计数直接修改原数组，返回被删除元素组成的数组。支持负索引，-1表示最后一个元素
- **slice()**：创建原数组的浅拷贝，接受起止索引参数返回新数组，不改变原数组
- **length属性**：JS数组的length属性可写，减小length会直接截断数组

### 常见误区

- 误认为修改length属性只是隐藏元素而非真实删除
- 混淆slice()的end参数为闭区间（实际是左闭右开）
- splice(start, deleteCount)中deleteCount参数为删除数量而非结束索引

## 问题解答

**splice()会修改原数组**，该方法通过直接操作数组内容实现增删元素；**slice()不修改原数组**，仅返回指定范围的浅拷贝数组。

三种删除末位元素的方法：

1. **pop()**：直接移除并返回最后一个元素

```javascript
const last = arr.pop(); // 修改原数组
```

2. **splice()**：使用负索引定位

```javascript
arr.splice(-1, 1); // 从倒数第一位开始删除1个元素
```

3. **length属性**：通过修改数组长度截断

```javascript
arr.length = Math.max(0, arr.length -1); // 防止负数长度
```

## 解决方案

### 编码示例

```javascript
// 方法1：pop()
function removeLastByPop(arr) {
  if (!Array.isArray(arr)) throw new TypeError();
  return arr.length ? arr.pop() : undefined;
}

// 方法2：splice()
function removeLastBySplice(arr) {
  return arr.splice(-1, 1)[0]; // 返回被删除元素
}

// 方法3：修改length
function removeLastByLength(arr) {
  const newLen = Math.max(0, arr.length -1);
  const removed = arr[newLen]; // 需提前保存返回值
  arr.length = newLen;
  return removed;
}
```

### 可扩展性建议

- 大数据量场景优先使用splice()，避免频繁修改length引发引擎重新分配内存
- 低版本浏览器需注意splice()的兼容性（IE9+）

## 深度追问

### 如何获取被删除元素的引用？

- pop()和splice()直接返回被删元素，length方式需预先存储arr[newLength]

### 哪种方式可能引发内存泄漏？

- 修改length方式可能导致游离的DOM元素引用，需手动置null

### 伪数组转换如何实现？

- 先用Array.from()转为真数组再操作，或使用call绑定数组方法
