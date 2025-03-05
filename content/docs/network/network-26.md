---
weight: 3600
date: '2025-03-04T09:31:00.147Z'
draft: false
author: zi.Yang
title: 同源策略与跨域解决方案
icon: public
toc: true
description: >-
  浏览器的同源策略如何限制跨域请求？请对比CORS（预检请求/简单请求）、反向代理、JSONP等方案的实现原理及安全性差异，说明何时应优先选择CORS的withCredentials凭证模式。
tags:
  - network
  - 同源策略
  - 跨域请求
  - 安全策略
---

## 考察点分析

该题目主要考察以下核心能力维度：

1. **浏览器安全机制理解**：同源策略的设计目标与具体限制维度
2. **跨域方案技术选型**：对比不同解决方案的底层原理与适用场景
3. **安全防护意识**：分析各方案的安全边界与风险点
4. **工程实践能力**：根据业务场景选择最佳跨域方案

具体技术评估点：

- 同源策略的三要素判定（协议/域名/端口）
- CORS预检请求触发条件与流程控制
- JSONP的脚本注入风险与防御措施
- 反向代理方案的服务器架构影响
- 凭证模式的CORS配置注意事项

---

## 技术解析

### 关键知识点

CORS > 反向代理 > JSONP > 同源策略

#### 原理剖析

1. **同源策略**：浏览器安全基座，限制跨源资源交互。影响XMLHttpRequest、Fetch API、Web Fonts、Web Workers等
   - 跨域写（Cross-origin writes）：默认允许（如表单提交）
   - 跨域资源嵌入（Cross-origin embedding）：需MIME类型校验
   - 跨域读（Cross-origin reads）：默认禁止

2. **CORS**：
   - 简单请求：满足特定条件（GET/POST/HEAD，Content-Type为text/plain等）
   - 预检请求：非简单请求先发OPTIONS验证（复杂请求特征检测）

   ```http
   Access-Control-Allow-Origin: https://example.com
   Access-Control-Allow-Methods: POST, GET
   Access-Control-Allow-Headers: X-Custom-Header
   ```

3. **JSONP**：利用`<script>`标签不受同源限制的特性

   ```javascript
   function handleResponse(data) {
     console.log('Received:', data);
   }
   const script = document.createElement('script');
   script.src = 'http://external.com/data?callback=handleResponse';
   document.body.appendChild(script);
   ```

4. **反向代理**：服务端中转实现同源访问

   ```nginx
   location /api/ {
     proxy_pass http://backend-server:8080/;
     proxy_set_header Host $host;
   }
   ```

#### 安全性差异对比

| 方案       | 安全风险                          | 防御措施                     |
|------------|---------------------------------|----------------------------|
| CORS       | 配置错误导致CSRF                | 严格设置allow-origin        |
| JSONP      | XSS攻击、回调劫持               | 输入过滤+随机回调名         |
| 反向代理   | 增加攻击面                      | 代理层请求过滤              |

---

## 问题解答

浏览器同源策略通过协议/域名/端口三要素校验阻止跨域资源访问，主要限制AJAX请求、DOM访问和存储隔离。CORS通过服务端响应头实现跨域授权，需区分简单请求（直接发送）与预检请求（OPTIONS预检）。JSONP利用脚本标签跨域特性但存在XSS风险，反向代理通过服务端中转隐藏跨域。withCredentials凭证模式应在前端需携带Cookies/HTTP认证且服务端配置Access-Control-Allow-Credentials: true时使用，同时需避免使用通配符(*)配置origin。

---

## 解决方案

### CORS配置示例（Node.js）

```javascript
const express = require('express');
const cors = require('cors');

const app = express();
app.use(cors({
  origin: 'https://client.com', // 精确控制允许源
  methods: ['GET', 'POST'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true // 启用凭证模式
}));

// 复杂请求处理
app.options('/data', (req, res) => {
  res.header('Access-Control-Max-Age', 86400); // 预检缓存24小时
  res.sendStatus(204);
});
```

### 可扩展建议

1. 高并发场景：预检请求缓存优化（Access-Control-Max-Age）
2. 多环境适配：通过环境变量动态配置允许源
3. 安全加固：配置CORS中间件白名单校验

---

## 深度追问

1. **CORS预检请求为什么能提升安全性？**
   - 强制服务端声明允许的跨域操作类型，防止滥用复杂请求

2. **JSONP如何防御恶意数据注入？**
   - 校验返回数据格式，强制JSON结构验证

3. **反向代理如何处理Cookie传递？**
   - 配置proxy_cookie_path实现域名转换
