---
weight: 3500
date: '2025-03-04T09:31:00.147Z'
draft: false
author: zi.Yang
title: 资源加载性能优化
icon: public
toc: true
description: >-
  对比preload/prefetch/dns-prefetch资源提示的适用场景，说明HTTP/2 Server
  Push如何减少RTT延迟，以及图片懒加载的IntersectionObserver实现方案。
tags:
  - network
  - 加载优化
  - 资源提示
  - HTTP/2
---

## 考察点分析

本题主要考核候选人对以下维度的理解：

1. **浏览器资源加载机制**：区分preload/prefetch/dns-prefetch的工作原理与应用场景
2. **HTTP协议演进认知**：解析HTTP/2 Server Push的RTT优化原理
3. **性能优化实践能力**：使用现代API实现懒加载的技术选型与实现细节

具体技术评估点：

- 资源提示的优先级控制（preload立即加载 vs prefetch空闲加载）
- DNS预解析与TCP连接建立的时序关系
- Server Push的主动推送机制与传统请求响应模型的差异
- IntersectionObserver替代传统滚动监听方案的优势
- 懒加载实现中的临界条件处理（占位符、加载失败等情况）

---

## 技术解析

### 关键知识点

HTTP/2 Server Push > 资源提示优先级 > IntersectionObserver API

### 原理剖析

**资源提示对比**：

- `preload`：强制浏览器立即请求指定资源（优先级：High），适用于当前导航立刻需要的资源（如首屏字体/关键CSS）
- `prefetch`：低优先级预加载未来页面可能使用的资源（如下一页的图片），浏览器空闲时执行
- `dns-prefetch`：提前解析跨域DNS，缩短后续请求的DNS查询时间（约20-120ms）

**HTTP/2 Server Push**：
通过服务端主动推送关联资源，减少客户端解析HTML后再次请求资源的RTT（Round Trip Time）。例如请求`index.html`时主动推送`style.css`，消除CSS文件请求的1个RTT（传统HTTP/1.1需要：HTML请求→HTML响应→CSS请求→CSS响应）。

**IntersectionObserver**：
通过异步观察目标元素与视窗交叉状态，避免传统滚动监听的高频计算。当图片进入视口时替换`data-src`为真实src，实现按需加载。

### 常见误区

- 误用preload加载非关键资源，导致关键资源被延迟
- 认为dns-prefetch能加速同域资源（实际同域DNS已缓存）
- 未配置Server Push缓存策略导致重复推送
- 懒加载未设置阈值（threshold）导致图片加载过早

---

## 问题解答

**资源提示场景**：

- `preload`：首屏关键字体、折叠上方CSS/JS
- `prefetch`：用户鼠标悬停时预取下一页商品数据
- `dns-prefetch`：电商平台预解析第三方支付域名DNS

**HTTP/2 Server Push**：
服务端在响应主资源时主动推送子资源，消除客户端"发现-请求"的等待时间。例如推送首屏CSS文件可减少至少1次RTT，但需配合缓存策略避免过度推送。

**IntersectionObserver懒加载**：

```javascript
// 观察所有带data-src的图片
const observer = new IntersectionObserver((entries) => {
  entries.forEach(entry => {
    if (entry.isIntersecting) { 
      const img = entry.target
      img.src = img.dataset.src
      observer.unobserve(img) // 加载后解除观察
    }
  })
}, {
  rootMargin: '200px', // 提前200px触发加载
  threshold: 0.01
})

document.querySelectorAll('img[data-src]').forEach(img => observer.observe(img))
```

---

## 解决方案

### 编码优化点

1. **加载失败处理**：添加`onerror`事件切换备用源
2. **占位策略**：使用低质量图片占位（LQIP）提升体验
3. **兼容方案**：添加IntersectionObserver polyfill或降级为滚动监听

### 扩展性建议

- **大流量场景**：结合CDN预解析（如`<link rel="preconnect">`）
- **低端设备**：降低观察阈值，采用渐进加载策略
- **SEO优化**：添加`<noscript>`标签保证爬虫可抓取

---

## 深度追问

1. **如何验证Server Push实际效果？**
   - 使用Chrome DevTools的Network面板查看"Initiator"列的"Push"标识

2. **preload字体资源需要注意什么？**
   - 必须设置`crossorigin`属性，即使同域

3. **IntersectionObserver相比scroll事件监听的优势？**
   - 帧率无关检测，避免布局抖动（Layout Thrashing）
