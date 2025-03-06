---
weight: 1003000
date: '2025-03-04T06:58:29.122Z'
draft: false
author: zi.Yang
title: src与href属性差异对比
icon: html
toc: true
description: 请从资源加载机制、应用场景和浏览器处理方式三个方面，对比HTML元素中src属性与href属性的核心区别，并举例说明两者的典型使用场景。
tags:
  - html
  - HTML属性
  - 资源加载
---

## 考察点分析

本题主要考察以下核心能力维度：

1. **HTML规范理解**：对标准属性的语义化掌握程度
2. **浏览器工作原理**：资源加载机制与渲染管线的关联
3. **应用场景判断**：不同属性在工程实践中的合理运用

具体技术评估点包括：

- 资源加载的阻塞行为差异
- 属性与HTML元素的从属关系
- 浏览器对两类资源的预处理差异
- 缓存策略与预加载机制
- 跨域处理方式的区别

## 技术解析

### 关键知识点

1. **语义区别**：`src`（source）表示内容替换，`href`（hypertext reference）表示资源关联
2. **加载机制**：`src`资源会作为文档组成部分加载，`href`资源作为附加依赖加载
3. **执行影响**：带`src`的脚本会阻塞DOM构建，`href`样式表仅阻塞渲染

### 原理剖析

当浏览器解析到`<script src>`时：

1. 暂停HTML解析
2. 下载并立即执行脚本
3. 恢复文档解析

而`<link href>`的处理：

1. 异步下载CSS文件
2. 不阻塞HTML解析
3. 但会阻塞渲染树的合成

### 常见误区

- 误认为所有`href`都是非阻塞资源
- 混淆`<img>`的`src`与背景图加载优先级
- 忽略`preload`等属性对加载策略的影响

## 问题解答

`src`与`href`的核心差异体现在：

1. **资源类型**  
`src`用于**替换型内容**（如`<script>`/`<img>`），浏览器会下载并替换元素内容；`href`用于**关联型资源**（如`<a>`/`<link>`），建立文档与外部资源的引用关系。

2. **加载机制**  
`src`资源会阻塞文档解析（如未加`async`的`<script>`），`href`资源异步加载但可能阻塞渲染（如CSS文件）。`<img>`的`src`触发即时加载，而`<link rel="stylesheet">`的`href`可能被延迟加载。

3. **典型场景**  

- `src`：`<script src="app.js">`（执行JS）、`<img src="logo.png">`（嵌入图片）
- `href`：`<a href="/about">`（页面导航）、`<link href="style.css">`（引用样式表）

## 解决方案

### 编码示例

```html
<!-- 正确使用src的场景 -->
<script src="lazy-load.js" defer></script>
<img src="placeholder.jpg" loading="lazy" alt="...">

<!-- 合理使用href的示例 -->
<link href="print.css" rel="stylesheet" media="print">
<a href="/pr-pdf" download="whitepaper.pdf">下载白皮书</a>
```

### 性能优化

1. **预加载关键资源**：`<link rel="preload" href="critical.css" as="style">`
2. **延迟非关键脚本**：使用`async`/`defer`属性优化`src`加载
3. **响应式图片**：结合`srcset`属性适配不同分辨率

## 深度追问

### 如何避免图片src导致的布局偏移？

- 使用`width`/`height`预设占位空间，配合`aspect-ratio`CSS属性

### 不同资源类型对preconnect的影响？

- 字体资源需提前建立DNS连接，CSS需预加载但无需preconnect
