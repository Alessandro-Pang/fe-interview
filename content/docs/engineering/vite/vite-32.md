---
weight: 10032000
date: '2025-03-05T10:37:25.980Z'
draft: false
author: zi.Yang
title: Vite开发服务器HTTPS配置
icon: icon/vite.svg
toc: true
description: 如何配置Vite开发服务器以支持HTTPS？请通过`server.https`选项说明如何加载SSL证书（如`key`和`cert`文件路径）？
tags:
  - vite
  - 开发服务器
  - 安全配置
  - HTTPS
---

## 考察点分析

**核心能力维度**：工具链配置能力、Node.js API运用、开发环境安全配置

1. **Vite配置体系理解**：是否熟悉vite.config.js结构及server配置项
2. **HTTPS服务配置**：掌握Node.js原生https模块的证书加载方式
3. **安全文件处理**：正确使用文件系统API读取证书文件
4. **开发环境调试**：了解自签名证书的浏览器兼容处理

---

## 技术解析

### 关键知识点

1. Node.js `https.createServer`选项格式
2. `fs.readFileSync`同步文件读取
3. Vite开发服务器配置结构

### 原理剖析

Vite开发服务器底层使用Connect中间件框架，当配置`server.https`时会透传参数给Node.js的`https.createServer`方法。证书文件需通过文件系统API加载为Buffer或字符串，配置示例：

```javascript
// 典型配置结构
import { defineConfig } from 'vite'

export default defineConfig({
  server: {
    https: {
      key: fs.readFileSync('path/to/key.pem'),
      cert: fs.readFileSync('path/to/cert.pem')
    }
  }
})
```

### 常见误区

1. **路径处理错误**：未使用`path`模块导致相对路径异常
2. **同步异步混淆**：在动态加载配置时错误使用异步读取
3. **证书格式误解**：将PEM格式密钥与CRT格式混淆

---

## 问题解答

配置Vite开发服务器HTTPS需在`vite.config.js`中设置`server.https`选项。通过Node.js的`fs.readFileSync`同步读取SSL证书文件，具体步骤如下：

1. 导入`fs`和`path`模块处理文件路径
2. 使用绝对路径加载证书文件避免路径错误
3. 将证书内容作为`key`和`cert`属性传入配置对象

```javascript
import { defineConfig } from 'vite'
import fs from 'node:fs'
import path from 'node:path'

export default defineConfig({
  server: {
    https: {
      // 使用path.resolve处理跨平台路径问题
      key: fs.readFileSync(path.resolve(__dirname, 'localhost-key.pem')),
      cert: fs.readFileSync(path.resolve(__dirname, 'localhost.pem'))
    }
  }
})
```

---

## 解决方案

### 编码示例

```javascript
// vite.config.js
import { defineConfig } from 'vite'
import fs from 'node:fs'
import path from 'node:path'

export default defineConfig({
  server: {
    port: 443, // 标准HTTPS端口
    https: {
      // 支持同时加载多个域名证书
      key: [
        fs.readFileSync(path.join(__dirname, 'ssl/domain1.key')),
        fs.readFileSync(path.join(__dirname, 'ssl/domain2.key'))
      ],
      cert: [
        fs.readFileSync(path.join(__dirname, 'ssl/domain1.crt')),
        fs.readFileSync(path.join(__dirname, 'ssl/domain2.crt'))
      ]
    }
  }
})
```

**优化建议**：

1. 使用环境变量动态加载不同环境证书
2. 添加NODE_EXTRA_CA_CERTS处理根证书链

---

## 深度追问

### 如何自动生成开发证书？
>
> 使用`mkcert`工具创建本地可信证书

### HTTPS代理如何处理跨域？
>
> 配置server.proxy实现API请求转发

### 如何禁用证书校验？
>
> 设置`server.https.ca`为空数组（不推荐）
