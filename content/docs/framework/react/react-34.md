---
weight: 4400
date: '2025-03-05T12:28:17.278Z'
draft: false
author: zi.Yang
title: Next.js核心功能与场景
icon: icon/react.svg
toc: true
description: >-
  Next.js的核心功能如SSR、SSG、API
  Routes分别适用于哪些场景？例如电商首页SEO优化选择SSR，文档站点使用SSG，如何通过`getServerSideProps`和`getStaticProps`配置？
tags:
  - react
  - Next.js
  - 服务端渲染
  - 静态生成
---

## 考察点分析

该题主要考察候选人对现代SSR框架的**场景化架构能力**和**渲染策略决策能力**，重点评估：

1. **渲染模式原理理解**：SSR(服务端渲染)与SSG(示例场景：商品详情页访问量100万/天)

## 技术解析

### 关键知识点

SSR > SSG > ISR > API Routes > 混合渲染策略

### 原理剖析

1. **SSR (Server-side Rendering)**

- 工作流程：请求到达时Node.js服务器执行`getServerSideProps`，获取实时数据生成完整HTML
- 适用场景：用户相关数据(如个性化推荐)、高频更新的资讯类页面、需要首屏性能但无法预生成的内容
- 性能消耗：TTFB时间受数据接口响应速度影响

2. **SSG (Static Site Generation)**

- 工作原理：构建阶段通过`getStaticProps`生成静态HTML，可托管CDN
- 强化模式：通过`revalidate`实现ISR（增量静态再生），在指定时间间隔后重新生成
- 适用场景：产品文档、营销落地页、电商商品详情页等相对静态内容

3. **API Routes**

- 本质：基于Vercel Serverless的BFF层(Backend for Frontend)
- 核心能力：直接访问数据库/第三方API，解决浏览器跨域问题
- 典型应用：身份验证、支付回调、敏感操作

### 常见误区

1. 在SSG页面中调用浏览器API（如localStorage）
2. 误将`getServerSideProps`用于可缓存内容导致服务器压力过大
3. 未处理SSG的`fallback`状态导致404体验不佳

## 问题解答

Next.js的渲染策略选择需综合考量内容时效性、流量规模和数据源特性：

1. **SSR适用场景**

- 电商首页：需要实时库存状态和个性化推荐
- 用户仪表盘：包含敏感数据的动态仪表盘
- 配置示例：

```javascript
export async function getServerSideProps(context) {
  const res = await fetch('https://api.com/realtime-data')
  return { props: { data: await res.json() } }
}
```

2. **SSG最佳实践**

- 文档网站：内容更新频率低于1次/小时
- 产品介绍页：营销活动页面
- 强化配置：

```javascript
export async function getStaticProps() {
  return { 
    props: { /* ... */ },
    revalidate: 3600 // 启用ISR，每小时再生
  }
}
```

3. **API Routes典型用例**

- 支付回调处理：`/api/payment-callback`
- 数据库直连：避免前端暴露连接字符串
- 代理层：封装第三方API调用

## 解决方案

### 混合渲染示例

```javascript
// 商品详情页（SSG+CSR）
export async function getStaticProps() {
  const product = await getProductFromCMS() // 基础数据预取
  return { props: { product }, revalidate: 60 }
}

function ProductPage({ product }) {
  // 客户端获取实时库存
  const [stock, setStock] = useState(product.stock)
  useEffect(() => {
    fetchStock(product.id).then(setStock)
  }, [])
  
  return <div>{stock}件剩余</div>
}
```

### 性能优化

1. **CDN缓存策略**：SSG页面设置Cache-Control: public, max-age=31536000
2. **流式SSR**：结合Suspense边界逐步发送HTML片段
3. **边缘计算**：通过Middleware实现按地域SSR

## 深度追问

1. **如何实现SSG页面的多语言支持？**

- 答案提示：基于`getStaticPaths`的locale参数生成静态版本

2. **SSR场景下如何避免服务器过载？**

- 答案提示：结合Redis缓存+请求限流

3. **如何监控SSR的TTFB指标？**

- 答案提示：使用`next-ax`或自定义APM埋点
