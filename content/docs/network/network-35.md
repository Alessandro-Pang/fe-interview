---
weight: 4500
date: '2025-03-04T09:31:00.148Z'
draft: false
author: zi.Yang
title: 预检请求触发条件与优化
icon: public
toc: true
description: >-
  哪些跨域请求会触发OPTIONS预检请求？如何通过Access-Control-Max-Age减少预检次数？列举简单请求（Simple
  Request）的严格条件。
tags:
  - network
  - 跨域请求
  - CORS
  - 性能优化
---

## 考察点分析

**核心能力维度**：跨域请求机制理解、HTTP协议规范掌握、性能优化实践能力  
**技术评估点**：  
1. 区分简单请求与预检请求的触发条件  
2. 掌握CORS预检缓存机制及优化手段  
3. 准确记忆简单请求的3个严格限制条件  
4. 理解浏览器安全策略与性能优化的平衡  

---

## 技术解析

### 关键知识点
CORS预检机制 > 简单请求判定条件 > Access-Control-Max-Age优化

### 原理剖析
浏览器执行跨域请求时，根据请求特征决定是否触发预检（Preflight）：  
1. **非简单请求**会触发OPTIONS预检，包括：  
   - 使用PUT/DELETE/PATCH方法  
   - 包含自定义请求头（如Authorization）  
   - Content-Type非以下值：  
     `text/plain` `multipart/form-data` `application/x-www-form-urlencoded`  

2. **Access-Control-Max-Age**响应头设置缓存时间（秒），使浏览器在有效期内复用预检结果，减少OPTIONS请求次数。该值过大会导致策略更新延迟，建议设置为合理时间（如2小时=7200）。

3. **简单请求**必须同时满足：  
   - 方法为GET/HEAD/POST  
   - 仅包含以下头：  
     `Accept` `Accept-Language` `Content-Language` `Content-Type`  
   - Content-Type为上述三个允许值

### 常见误区
- 误认为所有POST请求都是简单请求  
- 忽略Content-Type中字符编码的影响（如`application/json;charset=UTF-8`仍会触发预检）  
- 混淆预检缓存与实际请求缓存机制  

---

## 问题解答

**触发OPTIONS的条件**：  
1. 使用PUT/DELETE/PATCH方法  
2. 包含自定义请求头  
3. Content-Type非标准值（如application/json）  

**Access-Control-Max-Age优化**：  
设置响应头`Access-Control-Max-Age: 7200`，使2小时内同源请求跳过预检阶段。需注意服务端配置变更时客户端缓存未过期导致的策略失效问题。

**简单请求条件**：  
1. 方法限制：GET/HEAD/POST  
2. 头部限制：仅允许标准头集合  
3. Content-Type限制：特定MIME类型且无额外参数  

---

## 解决方案

```javascript
// 后端CORS配置示例（Node.js）
const express = require('express');
const app = express();

// 处理预检请求
app.options('/api', (req, res) => {
  res.header('Access-Control-Allow-Origin', '*')
     .header('Access-Control-Allow-Methods', 'GET,POST,PUT')
     .header('Access-Control-Allow-Headers', 'Content-Type,Authorization')
     .header('Access-Control-Max-Age', 7200) // 2小时缓存
     .send();
});

// 处理实际请求
app.post('/api', (req, res) => {
  res.header('Access-Control-Allow-Origin', '*').json({ data: 'ok' });
});
```

**复杂度优化**：  
- 时间复杂度：预检缓存使重复请求从O(n)降到O(1)  
- 空间复杂度：浏览器单次缓存约1KB策略数据  

**扩展建议**：  
- 高频接口：延长Max-Age至24小时  
- 敏感接口：缩短缓存时间至300秒  
- 移动端场景：结合请求合并减少预检次数  

---

## 深度追问

1. **预检请求是否会携带cookie**？  
答：仅当设置`credentials: include`时会携带  

2. **如何避免预检请求影响首屏速度**？  
答：使用`<link rel=preconnect>`预建立连接  

3. **JSONP能否绕过预检**？  
答：可以但存在安全风险，已不推荐使用