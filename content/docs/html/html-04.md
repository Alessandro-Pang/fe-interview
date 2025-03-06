---
weight: 1004000
date: '2025-03-04T06:58:29.122Z'
draft: false
author: zi.Yang
title: script标签加载策略
icon: html
toc: true
description: 请绘制async与defer属性的执行时序图，说明两者在脚本下载、文档解析和脚本执行阶段的差异，并给出不同场景下的最佳实践建议。
tags:
  - html
  - 脚本加载
  - 性能优化
---

## 考察点分析

本题主要考察以下核心能力维度：

1. **浏览器渲染机制**：理解脚本加载对文档解析的影响
2. **性能优化认知**：不同加载策略对首屏时间/LCP指标的影响
3. **工程化思维**：根据业务场景选择最佳加载方案

具体技术评估点：

- 脚本下载与文档解析的阻塞关系
- 执行时机对DOMContentLoaded事件的影响
- 多脚本场景下的执行顺序控制
- 网络条件与脚本依赖关系的协同处理

---

## 技术解析

### 关键知识点优先级

1. 渲染阻塞机制 > 执行时序控制 > 依赖管理

### 原理剖析

浏览器解析HTML时，默认同步加载脚本会触发：

1. 停止文档解析
2. 下载脚本（同步阻塞）
3. 立即执行脚本
4. 恢复文档解析

**async脚本**：

- 异步下载（不阻塞解析）
- 下载完成后**立即执行**（可能中断文档解析）
- 执行顺序不保证（先下载完的先执行）

**defer脚本**：

- 异步下载（不阻塞解析）
- 在DOMContentLoaded事件**前顺序执行**
- 严格保持脚本声明顺序

### 常见误区

1. 认为async/defer都能完全避免渲染阻塞（async执行仍可能阻塞）
2. 混淆执行顺序（async不保序，defer保序）
3. 忽视DOMContentLoaded事件触发时机

---

## 问题解答

**执行差异对比表**：

| 特性        | 默认   | async  | defer     |
|---------------|--------|--------|-----------|
| 阻塞解析       | 是     | 否     | 否        |
| 执行时机       | 立即   | 下载完 | 文档解析后 |
| 顺序保证       | N/A    | 不保证 | 声明顺序  |

**最佳实践**：

1. **关键渲染路径脚本**：使用`defer`保持执行顺序，避免阻塞首屏渲染
2. **独立第三方库**（如统计代码）：使用`async`实现非阻塞加载
3. **强依赖DOM的脚本**：必须用`defer`或放在body末尾
4. **现代模块化方案**：使用`type="module"`（默认defer行为）

---

## 解决方案

### 代码示例

```html
<!-- 关键业务逻辑：保证顺序且不阻塞 -->
<script defer src="app.js"></script>

<!-- 独立SDK：尽快执行不阻塞 -->
<script async src="analytics.js"></script>

<!-- 内联脚本需后置 -->
<script>
document.addEventListener('DOMContentLoaded', () => {
  // 安全操作DOM
})
</script>
```

### 可扩展性建议

1. **大流量场景**：配合preload实现优先级控制

```html
<link rel="preload" href="critical.js" as="script">
```

2. **低端设备**：动态注入脚本控制超时

```javascript
const script = document.createElement('script');
script.src = 'heavy.js';
script.onerror = handleFailure;
document.body.appendChild(script);
```

---

## 深度追问

1. **如何预加载async脚本但不立即执行？**
   使用`<link rel="preload">`配合`async`属性

2. **模块化脚本(type="module")的默认行为？**
   默认具有defer特性，可添加async覆盖

3. **动态插入的脚本默认是async吗？**
   是，需显式设置async=false来保持顺序
