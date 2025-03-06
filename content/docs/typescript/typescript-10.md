---
weight: 7010000
date: '2025-03-04T08:37:03.207Z'
draft: false
author: zi.Yang
title: 模块化类型导入导出
icon: icon/typescript.svg
toc: true
description: 如何在.ts文件中使用import/export语法实现类型共享？对比全局声明（declare global）与模块化声明的适用场景。
tags:
  - typescript
  - 模块化
  - 类型导出
  - 全局类型
---

## 考察点分析

该题目主要考察以下核心能力维度：

1. **TypeScript模块系统理解**：掌握ES Module在类型系统中的具体应用，区分模块作用域与全局作用域
2. **类型声明作用域管理**：辨析`declare global`与模块化导出在类型可见性、作用范围方面的差异
3. **工程化设计思维**：根据项目规模判断类型声明方式的合理性，理解类型隔离与复用的最佳实践

具体技术评估点包括：

- 模块化类型导入/导出语法规范
- 全局类型声明的作用域边界
- 声明合并(Declaration Merging)的应用场景
- 模块解析策略对类型可见性的影响
- 第三方库类型扩展的正确姿势

---

## 技术解析

### 关键知识点

1. ES Module类型系统 > 声明文件作用域 > 全局类型扩展
2. 模块解析策略 > 命名空间污染防范 > 类型可见性控制

### 原理剖析

在TypeScript中，使用`import/export`实现类型共享时：

- 导出端：通过`export interface`或`export type`显式导出类型
- 导入端：通过`import type`语法进行类型导入（推荐方式，避免运行时副作用）
- 编译器会建立模块依赖图，确保类型系统正确解析引用关系

全局声明(`declare global`)通过环境上下文声明全局可用类型：

- 仅在模块文件（含import/export）中使用`declare global`才有效
- 本质是通过声明合并机制扩展全局命名空间
- 适用于需要跨模块访问的基础类型或对内置类型的扩展

```typescript
// 模块化声明示例
// types.ts
export interface User {
  id: string;
  name: string;
}

// 使用方
import type { User } from './types';

// 全局声明示例
declare global {
  interface Window {
    SDK: ThirdPartySDK;
  }
}
```

### 常见误区

1. 在非模块文件中误用`declare global`（无import/export的.ts文件视为脚本，其顶层声明自动全局）
2. 混合使用模块导出与全局声明导致类型覆盖
3. 未使用`import type`导致打包后代码残留无用导入

---

## 问题解答

在.ts文件中使用模块化类型共享：

1. **导出类型**：使用`export`关键字显式导出类型定义
2. **导入类型**：推荐使用`import type`语法导入确保类型安全
3. **全局声明适用场景**：需扩展全局对象（如Window）或定义全项目通用类型时使用
4. **模块化优势**：保持类型作用域隔离，避免命名冲突，适合组件/功能模块的类型封装

**场景对比**：

- **全局声明**：适合小型工具库、浏览器环境扩展、跨多模块的基类类型
- **模块化声明**：适合中大型项目、组件库开发、需要严格作用域控制的场景

---

## 解决方案

### 编码示例

```typescript
// 模块化类型定义（userTypes.ts）
export type UserID = string & { readonly brand: unique symbol };

export interface UserProfile {
  id: UserID;
  name: string;
  lastLogin: Date;
}

// 全局扩展声明（global.d.ts）
declare global {
  interface Array<T> {
    stableSort(): T[];
  }
  
  type CurrencyCode = 'USD' | 'EUR' | 'CNY';
}

// 使用方（main.ts）
import type { UserProfile } from './userTypes';

const user: UserProfile = { /*...*/ };

// 全局类型直接使用
const prices: number[] = [5, 2, 8];
prices.stableSort(); // 自动识别扩展方法
```

### 可扩展性建议

1. **大型项目**：采用模块分层架构，通过`index.ts`进行类型再导出
2. **多环境适配**：通过条件类型扩展全局声明

   ```typescript
   declare global {
     interface Window {
       __DEV__?: boolean;
     }
   }
   ```

3. **性能优化**：使用`import type`确保类型代码零运行时开销

---

## 深度追问

1. **如何避免模块循环依赖导致类型解析失败？**
   - 使用`import type`消除运行时依赖，重构类型依赖图

2. **第三方库缺失类型时如何扩展？**
   - 创建`@types/package`目录，使用模块/全局声明补充类型

3. **模块声明文件(.d.ts)与常规.ts文件的区别？**
   - 声明文件仅包含类型声明，编译后会被删除，用于库的类型定义
