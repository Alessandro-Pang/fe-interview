---
weight: 2400
date: '2025-03-04T06:58:24.482Z'
draft: false
author: zi.Yang
title: 空对象检测与对象合并
icon: javascript
toc: true
description: 如何准确判断一个对象是否为空对象？请列举至少两种实现方法并说明其优缺点。同时请解释Object.assign()在合并多个对象时的属性覆盖规则。
tags:
  - javascript
  - 对象操作
---

## 考察点分析

本题考查候选人对JavaScript对象操作的深层理解，主要评估以下能力维度：

1. **对象属性检测机制**：准确区分对象自身属性与原型属性，理解可枚举性对检测的影响
2. **ES6+API掌握程度**：对Reflect、Object等新API的理解与应用场景判断
3. **对象合并原理**：Object.assign的浅拷贝特性与属性覆盖顺序规则

具体技术评估点：

- Object.keys与Reflect.ownKeys的差异
- JSON.stringify在对象检测中的陷阱
- Symbol属性对检测的影响
- 原型链属性遍历问题
- Object.assign的同名属性覆盖顺序

---

## 技术解析

### 关键知识点

1. 对象空值检测：`Object.keys` < `Reflect.ownKeys` < `JSON.stringify`
2. 属性覆盖规则：后源覆盖前源 > 同名属性合并 > 浅拷贝特性

### 原理剖析

空对象检测核心在于判断对象是否含有**自有属性**。`Object.keys()`仅返回可枚举的自有属性，但无法检测不可枚举属性（如通过`Object.defineProperty`设置的`enumerable:false`属性）和Symbol类型属性。`Reflect.ownKeys()`可获取所有自有属性键（包括Symbol），但需要ES6+环境。

JSON.stringify方案通过序列化结果判断，但存在严重缺陷：值为`undefined`、函数、循环引用等属性会被忽略，导致检测结果失真。

Object.assign合并时，按参数顺序依次复制源对象的**可枚举自有属性**，后者覆盖前者同名属性。对于引用类型属性执行**浅拷贝**，复制的是内存地址而非创建新对象。

### 常见误区

- 误用`for...in`检测时未过滤原型链属性
- 认为`Object.keys`能捕获所有自有属性
- 忽略Object.assign对Symbol属性的处理
- 混淆深拷贝与浅拷贝的区别

---

## 问题解答

### 空对象检测方法

1. **Object.keys()方案**

```javascript
function isEmpty(obj) {
  return Object.keys(obj).length === 0;
}
```

**优点**：代码简洁，浏览器兼容性好  
**缺点**：无法检测不可枚举属性和Symbol属性

2. **Reflect.ownKeys()方案**

```javascript
function isEmpty(obj) {
  return Reflect.ownKeys(obj).length === 0;
}
```

**优点**：检测所有自有属性类型  
**缺点**：需要ES6环境支持

### Object.assign覆盖规则

合并时按参数**从右向左**顺序覆盖同名属性，最终结果保留最后一个出现的属性值。对于嵌套对象执行**浅拷贝**，修改目标对象会影响源对象。

示例：

```javascript
const merged = Object.assign({a:1}, {a:2, b:{x:3}}, {b:{y:4}});
// 结果：{a:2, b:{y:4}} （b属性被整体替换）
```

---

## 深度追问

1. **如何检测包括原型链属性的空对象？**  
提示：递归检查__proto__链，需处理循环引用

2. **Object.assign与扩展运算符合并对象的区别？**  
提示：处理Symbol属性时的行为一致性

3. **深拷贝如何实现？**  
提示：structuredClone API与递归方案的取舍
