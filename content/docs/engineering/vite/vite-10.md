---
weight: 10010000
date: '2025-03-05T10:37:25.978Z'
draft: false
author: zi.Yang
title: Vite环境变量配置方法
icon: icon/vite.svg
toc: true
description: 如何在Vite中通过.env文件定义环境变量？请说明不同模式（开发/生产）下.env文件的优先级及如何通过`import.meta.env`访问变量？
tags:
  - vite
  - 环境变量
  - 配置管理
  - 安全性
---

## 考察点分析

本题主要考察候选人对Vite环境变量管理机制的理解，重点评估以下能力维度：

1. **工程化配置能力**：理解Vite多环境配置文件的命名规范与加载规则
2. **安全边界意识**：掌握客户端环境变量的暴露规则与敏感信息防护
3. **构建流程理解**：清楚不同构建模式下（开发/生产）环境变量的优先级逻辑
4. **运行时机制**：熟悉`import.meta.env`的底层实现原理

具体技术评估点：

- 环境变量文件命名规范（.env.[mode]）
- 多环境文件加载优先级逻辑
- VITE_前缀变量的注入机制
- 开发/生产模式下的变量隔离策略

## 技术解析

### 关键知识点

1. **环境文件加载顺序**：模式特定文件 > 通用文件
2. **变量暴露规则**：仅VITE_前缀变量注入客户端
3. **模式识别机制**：基于NODE_ENV自动识别开发/生产模式

### 原理剖析

Vite通过dotenv解析.env文件，加载顺序遵循：

```bash
.env.${mode}.local > .env.${mode} > .env.local > .env
```

（例：生产模式下`.env.production.local`优先级最高）

客户端通过`import.meta.env`访问变量时，Vite会：

1. 过滤非VITE_前缀变量
2. 将合法变量注入import.meta.env对象
3. 在构建时通过字符串替换方式硬编码到产物中

### 常见误区

1. 误将敏感信息放入无VITE_前缀的变量
2. 错误认为.env.local仅适用于特定模式
3. 混淆process.env（Node端）与import.meta.env（客户端）

## 问题解答

在Vite中配置环境变量：

1. **创建.env文件**：
   - 通用配置：`.env`
   - 开发模式：`.env.development`
   - 生产模式：`.env.production`
   - 本地覆盖：`.env.local`（被Git忽略）

2. **变量访问**：

   ```javascript
   // 仅暴露以VITE_开头的变量
   console.log(import.meta.env.VITE_API_URL)
   ```

3. **优先级规则**：

   ```text
   开发模式：.env.development.local > .env.development > .env.local > .env
   生产模式：.env.production.local > .env.production > .env.local > .env
   ```

## 解决方案

### 编码示例

```bash
# .env.production
VITE_API_URL=https://api.example.com
VITE_CDN_HOST=cdn.prod.com

# .env.development
VITE_API_URL=http://localhost:3000
```

```javascript
// 组件中使用
const config = {
  api: {
    baseURL: import.meta.env.VITE_API_URL,
    timeout: 5000
  }
}
```

### 安全建议

1. 敏感变量（如数据库密码）必须使用无VITE_前缀的变量
2. 通过GitIgnore排除.local文件
3. 使用服务器端渲染时进行二次验证

## 深度追问

1. **如何自定义构建模式？**
   - 使用`--mode`参数：`vite build --mode staging`
   - 对应加载`.env.staging`文件

2. **Vite配置文件中如何读取环境变量？**
   - 使用`loadEnv`方法加载原始环境变量

   ```javascript
   // vite.config.js
   export default defineConfig(({ mode }) => {
     const env = loadEnv(mode, process.cwd(), '')
     return { base: env.INTERNAL_API_SECRET }
   })
   ```

3. **如何防止前端暴露敏感信息？**
   - 严格分离前后端环境变量
   - 使用代理服务器转发API请求
   - 启用代码混淆与压缩
