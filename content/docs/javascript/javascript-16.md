---
weight: 3016000
date: '2025-03-04T06:58:24.482Z'
draft: false
author: zi.Yang
title: 数组方法分类与应用
icon: javascript
toc: true
description: 请分类列举JavaScript数组的常用原生方法，包括修改原数组的方法和返回新数组的方法，并说明splice()与slice()的核心差异。
tags:
  - javascript
  - 数组操作
---

## 回答

### 考察点分析

此题主要考察：

1. **原生API掌握程度**：对JavaScript数组方法体系化认知能力
2. **可变性理解**：区分方法是否改变原数组，这对函数式编程和避免副作用至关重要
3. **方法参数辨析**：特别是splice与slice的参数模式差异
4. **内存机制认知**：返回新数组时是否产生深拷贝

技术评估点：

- 可变方法（Mutator Methods）与不可变方法（Accessor Methods）分类能力
- splice的参数结构及操作模式
- slice的浅拷贝特性
- 方法返回值类型判断
- ES6新增数组方法的归类

### 技术解析

#### 关键知识点

可变方法 > 不可变方法 > 浅拷贝机制 > 参数模式差异

#### 原理剖析

1. **可变方法**直接修改数组内存引用，操作后数组地址不变但内容变化
2. **不可变方法**创建新内存空间，通过浅拷贝生成新数组
3. **splice()**通过起始索引、删除数量、插入元素三个维度操作数组
4. **slice()**通过起止索引（左闭右开）截取数组副本

#### 常见误区

- 误将sort()/reverse()当作不可变方法
- 混淆slice的第二个参数为长度而非终止索引
- 认为返回新数组的方法都是深拷贝
- splice返回值误认为是修改后的数组

### 问题解答

**数组方法分类**：

```javascript
// 修改原数组（Mutator Methods）
push()/pop()         // 尾部操作
unshift()/shift()    // 头部操作
splice()             // 任意位置增删
sort()/reverse()     // 排序反转
fill()/copyWithin()  // ES6+方法

// 返回新数组（Accessor Methods）
concat()             // 数组合并
slice()              // 截取副本
map()/filter()       // 遍历转换
flat()/flatMap()     // ES6+扁平化
```

**splice()与slice()差异**：

| 特性        | splice()                  | slice()                  |
|------------|---------------------------|--------------------------|
| 修改原数组   | ✅                        | ❌                       |
| 参数模式     | (start, deleteCount, ...items) | (start, end)             |
| 返回值       | 被删除元素组成的数组       | 新数组                   |
| 内存影响     | 直接操作原始数组           | 创建浅拷贝新数组         |
| 典型场景     | 原地增删改元素             | 获取数组片段副本         |

### 解决方案

#### 编码示例

```javascript
// splice() 示例：删除并插入
const arr = [1,2,3,4,5];
const removed = arr.splice(1, 2, 'a', 'b');
console.log(arr); // [1, "a", "b", 4, 5]
console.log(removed); // [2, 3]

// slice() 示例：获取子数组
const original = [{x:1}, {x:2}];
const sliced = original.slice(0,1);
sliced[0].x = 9; // 影响原数组元素
console.log(original[0].x); // 9（浅拷贝证明）
```

### 深度追问

1. **如何实现数组的深拷贝？**  
提示：JSON序列化、递归克隆、structuredClone API

2. **splice的删除数量为0时有什么应用场景？**  
提示：实现纯插入操作，如插入中间元素

3. **为什么说filter()可能造成内存浪费？**  
提示：全量遍历但可能仅需部分元素，可用惰性迭代器优化
