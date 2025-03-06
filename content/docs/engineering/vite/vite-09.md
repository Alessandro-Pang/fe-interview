---
weight: 10009000
date: '2025-03-05T10:37:25.978Z'
draft: false
author: zi.Yang
title: build与server配置项详解
icon: icon/vite.svg
toc: true
description: Vite的`build`和`server`配置项有哪些常见用法？例如如何配置生产构建的代码压缩、开发服务器的端口号或HTTPS支持？
tags:
  - vite
  - 构建配置
  - 开发服务器
  - 部署优化
---

## 考察点分析

该题考察候选人以下能力维度：

1. **构建工具深度使用**：是否掌握Vite核心配置项及其工程化应用
2. **构建优化意识**：能否根据场景选择合适的构建配置优化产物体积
3. **开发环境调优**：是否具备本地开发环境定制能力
4. **安全认知**：是否理解现代Web开发中的HTTPS配置要求

技术评估点：

- build.minify的压缩策略选择
- rollupOptions自定义打包配置
- server.proxy跨域解决方案
- HTTPS证书配置方式
- 预构建依赖配置逻辑

---

## 技术解析

### 关键知识点

1. 生产构建（build）
   - 代码压缩：`minify`（terser/esbuild）
   - 资源处理：`assetsInlineLimit`（base64内联阈值）
   - 输出控制：`outDir`/`sourcemap`/`manifest`
   - 高级配置：`rollupOptions`（覆盖Rollup配置）

2. 开发服务器（server）
   - 网络配置：`port`/`host`/`open`
   - 安全协议：`https`（自签名证书/自定义证书）
   - 代理系统：`proxy`（解决跨域/路径重写）
   - 热更新：`hmr`（心跳检测/协议配置）

```javascript
示例配置结构：
// vite.config.js
export default defineConfig({
  build: {
    minify: 'terser', // 压缩引擎选择
    rollupOptions: {
      output: { manualChunks: (id) => {...} } // 自定义代码分割
    }
  },
  server: {
    port: 3000, // 端口锁定
    https: { // HTTPS配置
      key: fs.readFileSync('key.pem'),
      cert: fs.readFileSync('cert.pem')
    }
  }
})
```

### 常见误区

1. 混淆开发/生产环境配置（如将server配置用于build）
2. 手动配置Rollup导致Vite默认优化失效
3. 误用HTTP/2协议导致代理失效
4. 未配置CORS策略直接访问第三方API

---

## 问题解答

Vite的配置系统通过`build`和`server`两大模块分别处理构建和开发环境：

**build配置要点：**

1. 代码压缩：`minify`支持`terser`（默认）或`esbuild`，后者速度更快但功能较少
2. 资源内联：`assetsInlineLimit`控制小于指定字节的图片转为base64
3. 分包策略：通过`rollupOptions.output.manualChunks`自定义代码分割规则
4. 产物分析：配合`rollup-plugin-visualizer`插件分析包体积

**server配置要点：**

1. 端口设置：`port`指定开发服务器端口，设置`strictPort: true`可防止自动端口变更
2. HTTPS支持：使用内置SSL证书（`https: true`）或自定义证书文件路径
3. 代理系统：`proxy`字段配置API代理，支持路径重写和WebSocket代理
4. 跨域处理：`cors`选项控制是否自动注入CORS头

---

## 解决方案

### 典型配置示例

```javascript
import { defineConfig } from 'vite'
import fs from 'fs'

export default defineConfig({
  build: {
    minify: 'terser',
    terserOptions: { compress: { drop_console: true } },
    rollupOptions: {
      output: {
        chunkFileNames: 'static/[hash].js',
        entryFileNames: 'static/[hash].js'
      }
    }
  },
  server: {
    port: 8080,
    https: { // 自定义证书
      key: fs.readFileSync('./cert/key.pem'),
      cert: fs.readFileSync('./cert/cert.pem')
    },
    proxy: {
      '/api': {
        target: 'https://api.example.com',
        changeOrigin: true,
        rewrite: path => path.replace(/^\/api/, '')
      }
    }
  }
})
```

### 优化建议

1. **代码压缩**：使用`build.css.minify`单独配置CSS压缩
2. **增量更新**：启用`build.watch`进行增量构建
3. **多环境配置**：通过环境变量区分开发/生产配置
4. **CDN适配**：通过`base`配置项适配CDN路径

---

## 深度追问

### 追问1：如何防止vendor.js体积过大？

**答**：通过`manualChunks`手动拆包或使用`splitVendorChunkPlugin`

### 追问2：如何实现构建产物的哈希命名？

**答**：配置`build.rollupOptions.output`的entry/chunk/asset文件名的`[hash]`占位符

### 追问3：如何处理旧浏览器的兼容性？

**答**：通过`build.target`指定ES语法版本，配合@vitejs/plugin-legacy插件打polyfill包
