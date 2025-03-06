---
weight: 7003000
date: '2025-03-04T08:37:03.206Z'
draft: false
author: zi.Yang
title: never类型的特殊用途
icon: icon/typescript.svg
toc: true
description: 说明never类型在抛出异常函数或无限循环函数中的类型推断规则。如何利用never类型实现穷尽性检查（Exhaustiveness checking）？
tags:
  - typescript
  - 类型推断
  - 控制流分析
  - 模式匹配
---

## 考察点分析

该题主要考察以下核心能力：

1. **类型系统理解**：对TypeScript中`never`类型的本质理解，包括其在类型系统中的特殊地位
2. **控制流分析**：理解编译器对函数执行路径的类型推断规则，特别是异常抛出和无限循环场景
3. **模式应用能力**：掌握类型守卫与穷举检查的结合使用，体现防御式编程思维

具体技术评估点：

- `never`类型在异常函数/循环函数中的自动类型推断机制
- 函数返回类型与控制流分析的交互关系
- 类型收窄（Type Narrowing）与可辨识联合（Discriminated Union）的配合
- 利用`never`实现编译时穷举检查的模式

## 技术解析

### 关键知识点

1. 返回不可达函数的类型推断
2. 控制流分析中的类型收窄
3. 类型兼容性规则的特殊情况

### 原理剖析

`never`类型表示永不存在值的类型，在类型系统中具有以下特性：

- 是任何类型的子类型（可赋值给所有类型）
- 没有类型是`never`的子类型（除了`never`自身）
- 在联合类型中会被自动过滤（`T | never → T`）

**异常函数推断**：当函数包含`throw`或`never`返回（如无限循环）时，TypeScript会将函数返回类型推断为`never`。编译器通过控制流分析识别无法到达的代码路径。

**穷举检查实现**：利用`never`在联合类型中的特性，通过将未处理的类型赋值给`never`变量触发类型错误。当处理完所有可能的联合类型成员后，default分支的变量类型应为`never`，若有遗漏则会产生类型不匹配。

### 常见误区

- 混淆`void`与`never`：`void`表示函数返回`undefined`，而`never`表示函数无法正常返回
- 错误认为所有未明确返回的函数都是`never`类型（需满足控制流分析条件）
- 忽略代码结构对类型推断的影响（如循环中的条件中断）

## 问题解答

在抛出异常或无限循环函数中，TypeScript会自动将返回类型推断为`never`，因为这类函数无法正常执行完成。进行穷举检查时，可利用`never`类型作为兜底分支的类型断言，当存在未处理的联合类型成员时，编译器会抛出类型错误。

示例：

```typescript
type Shape = { kind: 'Circle', radius: number } | { kind: 'Square', size: number };

function getArea(shape: Shape): number {
  switch(shape.kind) {
    case 'Circle':
      return Math.PI * shape.radius ** 2;
    case 'Square':
      return shape.size ** 2;
    default:
      // 若有未处理的类型，shape将无法赋值给never
      const _exhaustiveCheck: never = shape;
      return _exhaustiveCheck;
  }
}
```

当新增`Triangle`类型到`Shape`联合类型但未更新`switch`逻辑时，`default`分支的`shape`将具有`Triangle`类型，导致赋值给`never`的类型错误。

## 解决方案

### 编码示例

```typescript
// 定义可辨识联合类型
type NetworkState = 
  | { status: 'success', response: string }
  | { status: 'loading' }
  | { status: 'error', code: number };

function handleState(state: NetworkState): string {
  switch(state.status) {
    case 'success':
      return state.response;
    case 'loading':
      return 'Loading...';
    case 'error':
      return `Error: ${state.code}`;
    default:
      // 类型保护
      const _exhaustive: never = state;
      return _exhaustive;
  }
}
```

### 可扩展性建议

- 在大型项目中建议使用枚举标识类型状态
- 结合TSLint配置`switch-exhaustiveness-check`规则
- 对高频变更的联合类型进行抽象封装

## 深度追问

1. **如何区分`never`与`any`类型？**
   - `any`放弃类型检查，`never`表示逻辑不可能存在

2. **`never`类型在条件类型中的特殊作用？**
   - 在类型运算中作为过滤条件（如`T extends never ? A : B`）

3. **如何处理异步函数中的never类型？**
   - 异步函数返回`Promise<never>`表示永远不会resolve
