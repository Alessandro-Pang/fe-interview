---
weight: 2300
date: '2025-03-05T10:37:25.978Z'
draft: false
author: zi.Yang
title: Vite的Gzip与代码分割配置
icon: icon/vite.svg
toc: true
description: >-
  如何在Vite中启用Gzip压缩和代码分割（Code
  Splitting）？请结合`vite-plugin-compression`插件和Rollup配置说明优化方案？
tags:
  - vite
  - 性能优化
  - 代码分割
  - 压缩策略
---

## 考察点分析

- **核心能力维度**：工程化构建优化能力、Vite配置实践能力、Rollup底层原理理解
- **技术评估点**：
  1. 构建工具插件集成能力（vite-plugin-compression使用）
  2. Rollup代码分割配置实践（manualChunks策略）
  3. 压缩算法选择与阈值控制
  4. 构建产物优化策略（预压缩与动态压缩对比）
  5. 性能优化平衡点把控（请求数量vs缓存效率）

---

## 技术解析

### 关键知识点

1. `vite-plugin-compression` > Rollup代码分割 > 动态导入语法
2. **Gzip预压缩**：通过构建时预生成.gz文件，降低服务端实时压缩开销
3. **代码分割策略**：基于路由分割、依赖分析、模块复用率三种主流方案

### 原理剖析

Gzip压缩通过识别重复字节降低传输体积，Vite通过Rollup的`generateBundle`钩子插入压缩逻辑。代码分割通过Rollup的`output.manualChunks`配置实现模块分组，结合动态导入语法`import()`触发自动分割。

### 常见误区

1. 开发环境启用压缩（应仅在prod使用）
2. 分割粒度过于细碎（HTTP/2下仍需控制并发）
3. 忽略已有CDN压缩策略（需确认服务端配置）

---

## 问题解答

**Gzip压缩配置**：

1. 安装插件：`npm i vite-plugin-compression`
2. 配置阈值与算法：

```javascript
// vite.config.js
import viteCompression from 'vite-plugin-compression'

export default {
  plugins: [
    viteCompression({
      threshold: 10240, // 10KB以上文件进行压缩
      algorithm: 'gzip'
    })
  ]
}
```

**代码分割优化**：

1. 动态导入路由组件：

```javascript
// 自动触发代码分割
const Home = () => import('./views/Home.vue')
```

2. 自定义分割策略：

```javascript
// vite.config.js
export default {
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          if (id.includes('node_modules')) {
            if (id.includes('lodash')) {
              return 'vendor-lodash'
            }
            return 'vendor'
          }
        }
      }
    }
  }
}
```

---

## 解决方案

### 编码示例

```javascript
// 高级配置示例
import { defineConfig } from 'vite'
import viteCompression from 'vite-plugin-compression'

export default defineConfig({
  plugins: [
    viteCompression({
      verbose: true,
      disable: false,
      threshold: 10240,
      deleteOriginFile: false // 保留原始文件供未支持服务使用
    })
  ],
  build: {
    rollupOptions: {
      output: {
        manualChunks(id) {
          // 将超过50KB的npm包单独打包
          if (id.includes('node_modules')) {
            const lib = id.split('node_modules/')[1].split('/')[0]
            if (lib === 'react') return 'react-vendor'
            return lib.length > 20 ? 'misc' : lib
          }
        }
      }
    }
  }
})
```

### 扩展性建议

1. **大流量场景**：开启Brotli压缩（需配置`algorithm: 'brotliCompress'`）
2. **低端设备**：降低分割粒度避免内存溢出
3. **混合部署**：同时生成gzip与br格式，通过`Accept-Encoding`头自动适配

---

## 深度追问

1. **如何验证Gzip压缩效果？**
   - 使用Chrome DevTools的Network面板查看Content-Encoding响应头

2. **动态导入与静态导入分割差异？**
   - 静态导入由Rollup自动优化，动态导入创建新chunk

3. **如何避免vendor包过大？**
   - 配置manualChunks分割高频更新与稳定依赖
