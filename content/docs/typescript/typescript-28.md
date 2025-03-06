---
weight: 7028000
date: '2025-03-04T08:37:03.210Z'
draft: false
author: zi.Yang
title: Extract工具类型作用
icon: icon/typescript.svg
toc: true
description: Extract&lt;T, U&gt;如何提取联合类型T中可赋值给U的子类型？演示从多种事件类型中提取鼠标事件类型的场景
tags:
  - typescript
  - 类型提取
  - 联合筛选
  - 模式匹配
---

## 考察点分析

**核心能力维度**：  
本题考察对TypeScript类型系统高级特性的掌握程度，重点检验以下维度：

1. **工具类型原理理解**：能否准确解释Extract工具类型的工作机制
2. **类型兼容性判断**：对类型系统赋值兼容规则的掌握程度
3. **联合类型操作**：处理联合类型时的分布式条件类型特性
4. **实际场景应用**：将类型工具应用于具体业务场景的能力

**技术评估点**：  

- 条件类型（Conditional Types）的分布式特性  
- 类型兼容性（Type Compatibility）规则  
- 工具类型的实现原理  
- 浏览器事件类型的类型体系认知  

---

## 技术解析

### 关键知识点

1. **条件类型分布式特性**：`T extends U ? T : never`  
2. **类型兼容性**：结构化类型系统（Duck Typing）的赋值规则  
3. **工具类型实现**：`Extract<T, U>` 类型运算过程  

### 原理剖析

当使用`Extract<T, U>`时：

1. TypeScript会将联合类型`T`拆分为多个独立类型（分布式条件类型）
2. 对每个子类型`K`进行判断：`K`是否可赋值给`U`
3. 通过条件类型保留满足`K extends U`的子类型
4. 最终合并所有满足条件的类型形成新联合类型

```typescript
// 伪代码实现
type Extract<T, U> = T extends U ? T : never
```

### 常见误区

1. 误认为`Extract<1|'a', number>`结果为`number`类型（实际为`1`字面量类型）
2. 混淆`T extends U`与`U extends T`的方向判断
3. 忽略`never`类型在联合类型中的自动过滤特性

---

## 问题解答

**Extract工具类型**通过条件类型的分布式特性，筛选出联合类型`T`中所有能赋值给类型`U`的成员。其本质是通过`T extends U ? T : never`类型运算，将联合类型拆分为独立类型进行兼容性判断，保留符合要求的类型。

**场景演示**：

```typescript
// 定义浏览器事件类型联合
type EventTypes = 
  | MouseEvent 
  | KeyboardEvent
  | TouchEvent
  | FocusEvent;

// 提取鼠标相关事件类型
type MouseEvents = Extract<EventTypes, MouseEvent>;  // MouseEvent
```

---

## 解决方案

### 编码示例

```typescript
// 定义完整的浏览器事件类型体系
interface BaseEvent {
  readonly timeStamp: number
}

interface MouseEvent extends BaseEvent {
  x: number
  y: number
}

interface KeyboardEvent extends BaseEvent {
  key: string
}

// 联合类型包含多种事件
type AllEvents = MouseEvent | KeyboardEvent | Event;

// 类型提取：此处将正确提取出MouseEvent类型
type ExtractedMouseEvents = Extract<AllEvents, MouseEvent>;

// 测试用例
const handleMouseEvent = (e: ExtractedMouseEvents) => {
  console.log(`坐标：${e.x}, ${e.y}`)
};
```

**优化说明**：  

1. 类型运算时间复杂度O(n)，n为联合类型成员数量  
2. 通过接口继承保证类型结构兼容性  
3. 使用`Extract`替代手动过滤提升代码可维护性

---

## 深度追问

### 问题1：`Extract`与`Exclude`的核心区别？

**提示**：`Exclude`是取补集，`Extract`是取交集

### 问题2：如何提取所有包含特定属性的类型？

**提示**：使用条件类型`T extends { prop: infer P } ? T : never`

### 问题3：如何处理联合类型中的`never`？

**提示**：联合类型会自动过滤`never`，如`1 | never`等价于`1`
