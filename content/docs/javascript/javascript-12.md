---
weight: 3012000
date: '2025-03-04T06:58:24.482Z'
draft: false
author: zi.Yang
title: 对象拷贝方法对比
icon: javascript
toc: true
description: 请说明Object.assign()与对象扩展运算符的异同点，包括对setter方法的处理差异，并解释为什么它们都属于浅拷贝的实现方式？
tags:
  - javascript
  - 对象方法
  - 拷贝机制
---

## 考察点分析

本题主要考察面试者对JavaScript对象拷贝机制的理解深度，重点评估以下维度：

1. **浅拷贝核心概念**：能否准确区分浅拷贝与深拷贝的本质差异
2. **API特性对比**：对Object.assign与扩展运算符底层实现差异的掌握程度
3. **属性描述符处理**：对存取器属性（setter/getter）的特殊处理机制理解
4. **原型链处理**：是否了解两种方式对原型链属性的处理差异
5. **执行机制差异**：对赋值操作与属性定义两种不同复制机制的理解

## 技术解析

### 关键知识点

1. 赋值操作 vs 属性定义
2. 存取器属性处理
3. 原型继承差异

### 原理剖析

Object.assign通过赋值操作（`[[Set]]`）实现属性复制，会触发目标对象的setter方法。对象扩展运算符使用`Object.defineProperty`直接定义新属性，相当于创建数据描述符，不会触发setter。

浅拷贝的判定标准在于是否递归处理嵌套对象。两种方式在处理引用类型属性值时，都仅复制内存地址而非创建新对象。

### 常见误区

1. 误判扩展运算符为深拷贝
2. 混淆setter方法的触发场景
3. 忽略对Symbol属性的处理差异（Object.assign支持Symbol属性复制）
4. 错误认为扩展运算符可以复制原型链属性

## 问题解答

Object.assign与对象扩展运算符都通过复制第一层属性实现浅拷贝。核心差异在于：

1. **setter处理**：Object.assign通过赋值操作触发目标对象setter，扩展运算符直接定义新属性（相当于`Object.defineProperty`），不会触发setter
2. **原型属性**：Object.assign不复制原型链属性，扩展运算符保留原型链（通过`__proto__`）
3. **数组合并**：Object.assign支持多源对象合并，扩展运算符需嵌套使用实现相同效果

浅拷贝判定依据：两者在处理对象属性时，若属性值为引用类型，则复制其引用地址而非创建新对象。修改嵌套对象属性会同时影响源对象与拷贝对象。

## 解决方案

```javascript
// 创建包含setter的源对象
const source = {
  _secret: 42, // 伪私有属性
  get value() {
    return this._secret;
  },
  nested: { a: 1 }
};

// Object.assign示例
const target1 = {};
Object.assign(target1, source);
// target1.value变为42（数据属性）
// target1.nested与source.nested指向同一对象

// 扩展运算符示例
const target2 = { ...source };
// target2.value为42（数据属性）
// target2.nested与source.nested指向同一对象

// 性能优化：对于简单结构推荐扩展运算符（引擎级优化）
// 扩展性方案：可通过递归实现深拷贝函数
function deepClone(obj) {
  if (typeof obj !== 'object' || obj === null) return obj;
  const result = Array.isArray(obj) ? [] : {};
  for (const key in obj) {
    result[key] = deepClone(obj[key]);
  }
  return result;
}
```

## 深度追问

1. **如何实现setter的深拷贝？**
   - 通过Object.getOwnPropertyDescriptor获取属性描述符，递归处理value值

2. **Symbol属性的拷贝差异？*
   - Object.assign支持Symbol属性复制，扩展运算符需通过Object.getOwnPropertySymbols处理

3. **为何扩展运算符更推荐在React中使用？**
   - 不可变数据模式要求，避免引用污染，触发组件更新
