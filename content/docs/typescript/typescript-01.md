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
```

### 2. 函数参数与返回值

```typescript
// 函数返回值类型注解
function sum(a: number, b: number): number {
  return a + b;
}

// 箭头函数
const greet = (name: string): void => {
  console.log(`Hi, ${name}!`);
};
```

### 3.复杂类型组合

使用联合类型（|）、类型别名或接口：

```typescript
// 联合类型
let id: string | number = "ID-123";
id = 456;

// 类型别名
type User = {
  name: string;
  age?: number;  // 可选属性
};
const user: User = { name: "Bob" };
```

### 4. 类型断言

强制明确值的类型：

```typescript
const input: unknown = "123";
const num: number = parseInt(input as string);  // 断言为 string 类型

// 或使用尖括号语法（不推荐在JSX中使用）
const anotherNum: number = <number>input;
```

## 四、最佳实践

避免使用 any：尽量用更精确的类型（如 unknown）或联合类型替代。

开启严格模式：在 tsconfig.json 中设置 strict: true，强制处理 null/undefined。

优先用类型推断：如 let age = 30 隐式推断为 number，无需显式注解。
