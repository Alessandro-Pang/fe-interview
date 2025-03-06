---
weight: 7004000
date: '2025-03-04T08:37:03.206Z'
draft: false
author: zi.Yang
title: object与Object类型区别
icon: icon/typescript.svg
toc: true
description: 为什么说使用object类型比Object更符合类型安全原则？解释{}空对象类型与Object.prototype属性继承的关系。
tags:
  - typescript
  - 对象类型
  - 类型范围
  - 原型链
---

## 考察点分析

本题主要考察以下三个核心维度：

1. **TypeScript类型系统理解**：区分基础类型与对象类型的类型声明方式
2. **类型安全原则**：掌握object与Object在类型约束上的差异及其对类型安全的影响
3. **原型继承机制**：理解空对象类型`{}`与Object.prototype的关系

具体评估点包括：

- object类型与Object接口的本质区别
- 原始值包装对象对类型安全的影响
- 空对象类型的类型兼容性特征
- 原型链属性在类型检查中的表现

## 技术解析

### 关键知识点

object类型 > 原型继承 > 类型兼容性

#### 原理剖析

1. **object类型**（TS 2.2+）表示非原始类型的对象，包括：对象字面量、数组、函数等。排除string/number/boolean等原始类型
2. **Object接口**表示JavaScript的Object构造函数类型，包含所有对象（包括原始值包装对象）和奇怪的undefined（开启strictNullChecks时除外）
3. **空对象类型{}**是包含零个属性类型约束的对象类型，但仍继承Object.prototype的属性和方法

```typescript
// Object接受原始包装类型
const a: Object = new String('dangerous'); // 合法
// object拒绝原始类型
const b: object = {}; // 合法
const c: object = 'str'; // 类型错误

// {}类型可访问原型方法
const emptyObj: {} = {};
emptyObj.toString(); // 合法
```

#### 常见误区

- 误认为Object是所有对象的基类型（实际是any对象的接口）
- 混淆object与Object的约束范围
- 认为空对象类型`{}`无法访问任何方法

## 问题解答

**object vs Object**：

1. **类型安全**：object排除了原始类型，避免`let val: Object = 42`这种危险赋值
2. **原型继承**：空对象类型`{}`通过原型链继承Object.prototype的方法，尽管类型定义中无显式属性，类型检查器仍允许调用toString等原型方法
3. **类型范围**：Object包含所有对象和原始包装类型，object仅包含非原始类型对象

## 解决方案

```typescript
// 类型安全示例
function safeInput(param: object) {
  // 确保接收真实对象类型 
  console.log(param.toString());
}

// 危险示例：可能接受包装对象
function unsafeInput(param: Object) {
  // 允许传入new Number(42)等包装类型
  console.log(param.valueOf());
}

// 空对象类型演示
const emptyObj: {} = { id: 1 }; // 合法，类型兼容
// emptyObj.newProp = 1 // 错误：{}类型无此属性声明
```

**可扩展性建议**：

1. 库开发时优先使用object保证接口安全
2. 处理未知对象时用`{}`配合类型守卫
3. 低端设备注意Object.prototype方法的polyfill

## 深度追问

1. **如何检测一个变量是object类型？**
  使用`typeof val === 'object' && val !== null`

2. **为什么`Object.create(null)`无法调用toString？**
   该对象没有原型链继承，无法访问Object.prototype方法

3. **Record<string, any>与object的区别？**
   Record允许已知属性操作，object类型禁止属性直接访问
