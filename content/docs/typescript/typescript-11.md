---
weight: 2100
date: '2025-03-04T08:37:03.207Z'
draft: false
author: zi.Yang
title: 类成员可见性修饰符
icon: icon/typescript.svg
toc: true
description: >-
  TypeScript中public/private/protected修饰符如何控制类成员的访问权限？请说明protected成员在子类中的访问规则，以及private字段在运行时实际实现方式（如编译为WeakMap）
tags:
  - typescript
  - 面向对象
  - 访问控制
  - 编译策略
---

## 考察点分析

该题目主要考核以下核心能力维度：

1. **类型系统理解**：对TypeScript访问控制修饰符的语义及使用场景的掌握程度
2. **继承机制认知**：protected修饰符在类继承体系中的特殊作用域规则
3. **编译原理基础**：TypeScript类型擦除特性与私有字段的运行时实现机制

具体技术评估点：

- public/protected/private修饰符的访问边界
- 子类方法中访问protected成员的合法性边界
- private字段的编译策略差异（TS私有声明 vs ES6#私有字段）
- 类型擦除对运行时的影响
- 访问修饰符与设计模式的应用关系

---

## 技术解析

### 关键知识点

访问修饰符 > 继承可见性 > 编译策略

#### 原理剖析

TypeScript通过访问修饰符实现**编译阶段的访问控制**：

- `public`：默认修饰符，成员在任何位置可见
- `protected`：允许类内部及子类访问，禁止外部直接访问
- `private`：仅在声明类内部可见

**protected陷阱**：子类可通过`this`访问父类protected成员，但无法通过**子类实例**直接访问（需通过子类暴露的公共方法）

**private实现差异**：

- TS传统模式：通过`WeakMap`存储私有属性（每个实例对应独立存储）
- ES6#语法：编译为**硬编码私有标识符**（通过`Object.defineProperty`实现）

```javascript
// TS编译结果（target=ES5）
var _name = new WeakMap();
class A { 
  constructor() { _name.set(this, 'test'); }
}
```

#### 常见误区

1. 误以为TS的`private`在运行时完全不可访问（实际可通过`instance['_name']`绕过）
2. 混淆protected成员在子类中的访问方式（允许通过`this`访问，但禁止通过实例属性访问）
3. 忽略TS配置对私有字段编译策略的影响（如`useDefineForClassFields`参数）

---

## 问题解答

TypeScript通过访问修饰符实施**编译时访问控制**：

- `public`成员可被任意访问
- `protected`允许类及其子类内部访问，但禁止通过实例直接调用
- `private`仅在声明类内部可见，子类亦无法访问

**protected子类规则**：子类方法可通过`this`访问父类protected成员，但外部代码通过子类实例访问protected成员仍被禁止

**private运行时实现**：

- 当编译目标为ES5及以下时，使用`WeakMap`存储私有字段（每个实例关联独立存储空间）
- 当启用ES6+的`#`语法时，编译为JavaScript原生私有字段（通过闭包与唯一标识符实现硬隔离）

---

## 深度追问

1. **如何强制实现运行时私有？**
   - 使用ES6#语法或立即执行函数+闭包

2. **protected成员可否在子类中重定义为public？**
   - 允许，通过子类重新声明为public成员

3. **TS private与#字段编译差异？**
   - TS private仅类型检查，#字段生成实际私有属性
