---
weight: 4300
date: '2025-03-05T10:37:25.980Z'
draft: false
author: zi.Yang
title: Vite全局常量定义方法
icon: icon/vite.svg
toc: true
description: 如何在Vite中注入全局常量（如`__VERSION__`）？请使用`define`配置项说明如何替换代码中的占位符变量？
tags:
  - vite
  - 全局变量
  - 配置注入
  - 编译替换
---

## 考察点分析

本题主要考核候选人对前端工程化工具链的掌握程度，具体评估以下维度：

1. **编译时替换机制**：理解Vite的define配置本质是编译期的字符串替换
2. **环境变量处理**：区分import.meta.env与编译常量的使用场景
3. **类型安全**：处理全局常量时的TypeScript支持方案
4. **安全实践**：避免XSS攻击的JSON.stringify正确用法
5. **多环境适配**：不同编译模式下常量值的动态配置

## 技术解析

### 关键知识点

1. Vite的define配置 > 环境变量 > 类型声明文件
2. 编译时替换与运行时注入的本质区别
3. 字符串替换的安全处理方式

### 原理剖析

Vite的define配置通过esbuild的define功能实现编译时替换。在代码压缩阶段，所有匹配的标识符会被直接替换为指定的JSON值。例如配置`define: { __VERSION__: '"1.0.0"' }`，代码中的`__VERSION__`会被替换为字符串"1.0.0"。

**常见误区**：

- 直接赋值非JSON序列化值（如`__VERSION__: 1.0.0`）导致替换后出现未定义标识符
- 混淆import.meta.env（环境变量）与编译时常量的使用场景
- 未声明TypeScript类型导致类型检查报错

## 问题解答

在Vite中通过`define`配置项注入全局常量：

```javascript
// vite.config.js
export default defineConfig({
  define: {
    __VERSION__: JSON.stringify('1.0.0'), // 正确序列化字符串
    'import.meta.env.APP_MODE': JSON.stringify(process.env.NODE_ENV)
  }
})
```

该配置会在编译阶段将代码中的`__VERSION__`直接替换为`"1.0.0"`，同时确保字符串类型的正确性。对于TypeScript项目，需添加类型声明：

```typescript
// global.d.ts
declare const __VERSION__: string;
```

## 解决方案

### 编码示例

```javascript
// 安全注入含敏感信息的常量
const pkg = require('./package.json')

export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(pkg.version), // 动态读取版本号
    __BUILD_TIMESTAMP__: Date.now().toString(), // 编译时间戳
    __FEATURE_FLAGS__: JSON.stringify({
      experimental: process.env.DEBUG === 'true'
    })
  }
})
```

**优化建议**：

1. 通过环境变量区分编译模式（开发/生产）
2. 对高频访问的常量使用Object.freeze进行冻结
3. 复杂数据建议封装为JSON对象减少替换次数

## 深度追问

1. **如何防止常量被意外修改？**
   - 使用Object.freeze进行对象冻结，或在Proxy层拦截修改操作

2. **动态常量如何实现热更新？**
   - 编译时常量需重启服务，动态值建议使用状态管理或runtime环境变量

3. **替换后源码如何验证？**
   - 通过`vite build --mode development`保留源码映射，或使用AST分析工具检查
