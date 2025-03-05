---
weight: 4800
date: '2025-03-04T06:58:34.332Z'
draft: false
author: zi.Yang
title: 视口单位动态计算
icon: css
toc: true
description: >-
  请分析vh/vw单位在移动端浏览器地址栏可见性变化时的表现问题，说明dvh（dynamic viewport
  height）单位的解决原理，并演示使用@supports实现渐进增强的视口单位降级方案。
tags:
  - css
  - 移动端
  - 响应式
---

## 考察点分析

本题主要考核以下核心能力维度：

1. **移动端适配经验**：理解视口单位在动态视口场景下的痛点
2. **CSS新特性掌握**：dynamic viewport units（动态视口单位）的机制与应用
3. **渐进增强方案设计**：使用@supports实现兼容性方案的能力

具体技术评估点：

- 传统vh单位在移动端布局中的缺陷表现
- dvh单位的动态计算原理与浏览器支持情况
- CSS特性检测的工程化应用
- 视口变化事件的浏览器兼容性处理

---

## 技术解析

### 关键知识点

动态视口单位 > CSS特性检测 > 视口元标签 > JavaScript补偿方案

### 原理剖析

1. **传统vh问题**：移动端浏览器地址栏的显隐会改变`visual viewport`高度，但vh单位基于`layout viewport`计算，导致高度计算不更新（iOS 15之前尤为明显）

2. **dvh解决方案**：

   ```bash
   dvh = 当前可视窗口高度 - 浏览器UI占位高度
   ```

   浏览器实时计算可见视口区域，通过`VisualViewport`API动态更新值（需浏览器支持）

3. **@supports逻辑**：

   ```css
   /* 特性检测优先使用dvh */
   @supports (height: 100dvh) {
     .box { height: 100dvh; }
   }
   /* 降级方案 */
   @supports not (height: 100dvh) {
     .box { height: 100vh; }
   }
   ```

### 常见误区

- 误以为所有移动浏览器都会自动处理视口高度变化
- 混淆`svh`（小视口高度）与`dvh`的应用场景
- 忽视Safari 15.4之前版本需要`viewport-fit=cover`的配合

---

## 问题解答

移动端浏览器地址栏显隐会改变实际可视区域，但传统vh单位基于初始视口高度计算。当地址栏隐藏时，页面底部可能出现空白或被遮挡。dvh单位通过动态跟踪可视区域高度解决该问题，其值随浏览器UI变化实时更新。

渐进增强方案通过`@supports`进行特性检测，优先使用dvh，降级方案结合vh与JS动态计算。需配合`viewport-fit=cover`处理全面屏设备，并通过防抖优化resize事件。

---

## 解决方案

### 编码示例

```html
<!DOCTYPE html>
<html style="--dvh: 100vh"> <!-- 默认值 -->
<head>
  <meta name="viewport" content="viewport-fit=cover">
  <style>
    .fullscreen {
      /* 现代浏览器方案 */
      height: 100dvh;
      /* 兼容方案 */
      height: var(--dvh, 100vh);
    }

    /* 特性检测 */
    @supports (height: 100dvh) {
      :root { --dvh: 100dvh; }
    }
  </style>
</head>
<body>
  <div class="fullscreen"></div>
  <script>
    // 补偿方案
    if (!CSS.supports('height', '100dvh')) {
      const updateHeight = () => {
        document.documentElement.style.setProperty('--dvh', `${window.innerHeight}px`);
      };
      
      // 防抖处理
      let timer;
      window.addEventListener('resize', () => {
        clearTimeout(timer);
        timer = setTimeout(updateHeight, 100);
      });
      updateHeight();
    }
  </script>
</body>
</html>
```

### 可扩展性建议

1. 低端设备：使用CSS变量减少JS操作
2. 大流量场景：服务端根据User-Agent注入不同CSS
3. 高频变化场景：使用CSSOM直接操作样式

---

## 深度追问

### 追问1：如何检测浏览器是否支持dvh？

通过`CSS.supports('height', '100dvh')`进行特性检测，返回布尔值判断支持状态。

### 追问2：visual viewport与layout viewport区别？

`visual viewport`是当前可见区域，`layout viewport`是页面布局的基准容器，两者差异导致移动端滚动效果。

### 追问3：dvh的polyfill实现原理？

通过`window.visualViewport`API监听缩放和滚动事件，动态更新CSS自定义属性值。
