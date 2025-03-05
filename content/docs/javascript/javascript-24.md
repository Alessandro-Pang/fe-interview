---
weight: 3400
date: '2025-03-04T06:58:24.483Z'
draft: false
author: zi.Yang
title: 请求库特性对比
icon: javascript
toc: true
description: >-
  从API设计、错误处理、请求取消、浏览器兼容性等方面，对比原生AJAX、axios和fetch
  API的核心差异，并说明为什么现代项目更倾向于使用axios库？
tags:
  - javascript
  - 网络请求
  - 工具库
---

## 考察点分析

本题主要考察候选人：
1. **框架原理理解**：对HTTP请求库核心功能的实现机制理解
2. **技术方案对比**：多维度技术选型分析能力
3. **工程化思维**：现代前端项目的架构设计考量

具体评估点：
- 不同请求方案的API设计范式差异
- 错误处理机制的完整性对比
- 请求取消方案的技术实现
- 浏览器兼容性处理策略
- 现代框架的工程化需求匹配度

---

## 技术解析

### 关键知识点
1. 请求生命周期管理 > 错误边界处理 > API封装复杂度
2. XHR vs Fetch底层差异
3. Promise封装策略

### 核心差异对比
| 特性            | XHR               | fetch               | axios               |
|----------------|-------------------|---------------------|---------------------|
| API设计         | 回调驱动          | Promise基础         | Promise链式+拦截器  |
| 错误处理        | 手动检测状态码    | 仅网络错误reject     | 全自动HTTP错误处理  |
| 请求取消        | xhr.abort()       | AbortController     | CancelToken/Abort   |
| JSON处理        | 手动转换          | 手动json()方法      | 自动转换            |
| 浏览器兼容      | IE9+              | 现代浏览器           | XHR垫片方案         |

### 典型误区
1. 认为fetch会自动处理HTTP错误（实际仅处理网络层错误）
2. 混淆AbortController与CancelToken的适用场景
3. 忽视XHR的进度事件优势（上传/下载进度监控）

---

## 问题解答

原生AJAX基于XHR对象实现，采用事件回调机制，API设计较为冗余。fetch作为浏览器原生API引入Promise，但存在三大痛点：1）400/500状态码不会触发reject 2）默认不带cookie 3）缺乏请求取消原生支持。axios通过封装XHR提供更完善的解决方案：链式API、请求拦截、自动转换JSON、统一错误处理和CSRF防御。现代项目倾向axios因其在工程化需求（类型系统、测试友好性、可扩展性）方面表现更优，且能通过适配器同时支持浏览器/Node.js环境。

---

## 解决方案

### 核心代码示例
```javascript
// 错误处理对比示例
// XHR
const xhr = new XMLHttpRequest()
xhr.onload = () => {
  if (xhr.status >= 200 && xhr.status < 300) {
    console.log(JSON.parse(xhr.response))
  } else {
    console.error('请求失败', xhr.status)
  }
}

// fetch
fetch(url)
  .then(res => {
    if (!res.ok) throw new Error(res.status)
    return res.json()
  })
  .catch(handleNetworkError)

// axios
axios.get(url)
  .then(res => console.log(res.data))
  .catch(handleAllError) // 自动包含HTTP错误
```

### 可扩展性建议
1. **多环境适配**：通过axios的适配器实现SSR同构请求
2. **大流量优化**：配合拦截器实现请求节流、重试机制
3. **类型安全**：配合TypeScript定义统一响应结构

---

## 深度追问

1. **如何实现全局请求拦截？**
   - 答案提示：axios.interceptors.request.use()

2. **怎样处理CSRF防护？**
   - 答案提示：axios默认携带XSRF-TOKEN cookie

3. **如何监控请求进度？**
   - 答案提示：axios配置onUploadProgress回调