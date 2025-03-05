---
weight: 4000
date: '2025-03-04T08:37:03.210Z'
draft: false
author: zi.Yang
title: NonNullable工具类型作用
icon: icon/typescript.svg
toc: true
description: NonNullable&lt;T&gt;如何过滤掉类型T中的null和undefined？结合GraphQL非空字段校验，说明该工具类型的数据净化作用
tags:
  - typescript
  - 空值过滤
  - 数据净化
  - 非空断言
---

## 考察点分析

该题目主要考察以下核心能力维度：

1. **TypeScript 高级类型理解**：对工具类型的实现机制及使用场景的掌握
2. **类型安全实践**：结合具体应用场景（GraphQL）处理数据边界问题
3. **类型系统与运行时关联**：理解类型擦除特性及如何保证类型安全

具体技术评估点：

- NonNullable 工具类型实现原理
- 联合类型(Union Types)的分布条件类型处理
- GraphQL 非空校验与类型系统的协同机制
- 编译时类型检查与运行时数据校验的关系

---

## 技术解析

### 关键知识点

1. **条件类型(Conditional Types)**：`T extends U ? X : Y`
2. **分布式条件类型**：对联合类型的自动展开特性
3. **类型净化(Type Sanitization)**：通过类型操作保证数据契约

### 原理剖析

TypeScript 内置的 `NonNullable<T>` 通过条件类型排除空值：

```typescript
type NonNullable<T> = T extends null | undefined ? never : T
```

当 `T` 为联合类型时，条件类型会触发**分布式特性**，分别对每个成员进行判断。例如：

```typescript
type T1 = string | null | undefined
type T2 = NonNullable<T1> // string
```

执行过程相当于：

```typescript
(null extends null | undefined ? never : null) | // never
(undefined extends null | undefined ? never : undefined) | // never
(string extends null | undefined ? never : string) // string
```

最终结果为 `never | never | string` 简化为 `string`

### 常见误区

- **误解类型擦除**：认为类型声明能改变运行时数据（需配合运行时校验）
- **过度信任后端数据**：GraphQL 类型声明可能因实现错误返回非法值
- **滥用非空断言**：使用 `!` 操作符绕过类型检查可能导致运行时错误

---

## 问题解答

`NonNullable<T>` 通过条件类型排除 `T` 中的 `null` 和 `undefined`。在 GraphQL 场景中，当字段声明为 `String!` 时，前端可使用该类型进行二次验证：

```typescript
type GraphQLResponse = {
  user?: {
    name: string | null
    email!: string // GraphQL 非空声明
  }
}

type SafeUser = {
  name: NonNullable<GraphQLResponse['user']>['name'] // string
  email: NonNullable<GraphQLResponse['user']>['email'] // string
}
```

此时即使后端错误返回 `null`，类型系统仍会给出错误提示，强制进行空值处理。

---

## 解决方案

### 编码示例

```typescript
// 数据层响应类型
type UserResponse = {
  id: string | null
  name: string
  email?: string | null
}

// 安全转换函数
function sanitizeUser(user: UserResponse): NonNullable<UserResponse> {
  if (!user.id) {
    throw new Error("Invalid user data: id is required")
  }
  
  return {
    id: user.id, // TS 报错: Type 'string | null' 不能赋值给 'string'
    name: user.name,
    email: user.email ?? 'default@domain.com'
  } as NonNullable<UserResponse>
}
```

### 可扩展性建议

- **大流量场景**：将类型校验逻辑封装到 API 中间件统一处理
- **低端设备**：使用渐进增强策略，优先保证基础类型校验
- **错误监控**：集成 Sentry 等工具捕获类型断言异常

---

## 深度追问

1. **如何实现深度 NonNullable？**
提示：递归映射类型遍历对象属性

2. **GraphQL Codegen 如何自动生成 NonNullable 类型？**
提示：通过类型映射自动添加修饰符

3. **NonNullable 与 Type Guard 的区别？**
提示：编译时与运行时校验差异
