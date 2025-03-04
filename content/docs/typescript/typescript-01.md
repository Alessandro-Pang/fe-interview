---
weight: 1100
date: "2025-03-04T08:37:03.202Z"
draft: false
author: "zi.Yang"
title: "TypeScript基础类型种类"
icon: "icon/typescript.svg"
toc: true
description: "列举TypeScript中常用的原始数据类型（如string/number/boolean）及其特殊类型（any/unknown）。如何通过类型注解明确定义变量类型？"
tags: ["typescript", "基础类型", "类型注解", "类型安全"]
---

## 解答

# TypeScript 常用数据类型与类型注解指南

## 一、原始数据类型

| 类型           | 描述                                     | 示例                          |
|----------------|----------------------------------------|-------------------------------|
| `string`       | 字符串类型，表示文本数据                   | `let name: string = "Alice";` |
| `number`       | 数值类型（含整数、浮点数、二进制等）         | `let age: number = 25;`       |
| `boolean`      | 布尔类型，`true` 或 `false`              | `let isDone: boolean = false;`|
| `null`         | 表示“无值”，需在严格模式下使用             | `let data: null = null;`      |
| `undefined`    | 表示未初始化或未定义的变量                 | `let value: undefined = undefined;` |
| `symbol`       | 唯一且不可变的值（ES6+）                 | `const key: symbol = Symbol();` |
| `bigint`       | 大整数类型（后缀为 `n`）                 | `let big: bigint = 1000n;`    |

---

## 二、特殊类型

| 类型          | 描述                                                                 | 示例                                      |
|-------------|--------------------------------------------------------------------|------------------------------------------|
| `any`       | 禁用类型检查，允许赋值任何类型（慎用）                             | `let dynamic: any = "Hello"; dynamic = 1;` |
| `unknown`   | 安全的顶层类型，需类型校验后才能操作                               | `let val: unknown = fetchData(); if (typeof val === "string") { ... }` |
| `void`      | 表示函数无返回值（默认返回 `undefined`）                          | `function log(s: string): void { console.log(s); }` |
| `never`     | 表示永不返回值的函数（如抛出异常或死循环）                         | `function error(msg: string): never { throw new Error(msg); }` |

---

## 三、类型注解定义方式

### 1. 变量声明
```typescript
let count: number = 10;
const message: string = "Hello, TypeScript!";

