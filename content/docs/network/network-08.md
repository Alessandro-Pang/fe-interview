---
weight: 1800
date: '2025-03-04T09:31:00.136Z'
draft: false
author: zi.Yang
title: 强缓存与协商缓存机制
icon: public
toc: true
description: >-
  通过Cache-Control/Expires和Last-Modified/ETag两组头部，说明强缓存与协商缓存的触发条件及验证流程。为何推荐使用immutable指令优化缓存策略？
tags:
  - network
  - HTTP缓存
  - 缓存策略
  - 性能优化
---

## 考察点分析

**核心能力维度**：浏览器缓存机制理解、HTTP协议掌握程度、性能优化实践能力  

1. **缓存策略区分**：辨别强缓存与协商缓存的触发条件及交互流程  
2. **HTTP头部理解**：解析Cache-Control/Expires与Last-Modified/ETag的工作机制与优先级  
3. **性能优化思维**：评估immutable指令对缓存策略的改进价值及应用场景  
4. **工程实践认知**：缓存策略在CDN部署、版本管理等场景的应用考量  
5. **问题排查能力**：识别常见的缓存配置错误及验证逻辑漏洞  

---

## 技术解析

### 关键知识点  

Cache-Control > ETag > Last-Modified > Expires  

#### 原理剖析  

1. **强缓存**  

- 通过`Cache-Control: max-age=N`或`Expires`判断资源新鲜度  
- 资源未过期时直接返回200 OK (from disk/memory cache)，**不产生网络请求**  
- 优先级：`Cache-Control` > `Expires`（HTTP/1.1规范明确）

2. **协商缓存**  

- 强缓存失效后，携带`If-Modified-Since`（对应Last-Modified）或`If-None-Match`（对应ETag）发起请求  
- 服务器通过对比时间戳（Last-Modified）或内容哈希（ETag）判断资源变更  
- 未变更返回304 Not Modified，**更新本地缓存有效期**  

3. **Immutable指令**  

- `Cache-Control: immutable`声明资源内容永不变更  
- 有效期內**跳过协商验证**，彻底避免304请求，适用于带哈希版本号的静态资源  

#### 常见误区  

- 混淆`no-cache`（强制协商验证）与`no-store`（禁止缓存）  
- 认为ETag可完全替代Last-Modified（实际可并行使用）  
- 忽略304响应仍需要更新`Cache-Control`时效  
- 在动态资源错误使用immutable导致更新失效  

---

## 问题解答  

浏览器缓存通过两级策略提升性能：  

1. **强缓存**：  

- 检查`Cache-Control`/`Expires`，未过期直接使用本地缓存  
- 典型场景：重复访问静态资源（如图片、CSS）  

2. **协商缓存**：  

- 资源过期后，携带`If-Modified-Since`或`If-None-Match`发起请求  
- 服务器对比资源标识，未变更返回304，节省带宽  

**推荐immutable指令原因**：  

- 彻底消除协商请求，尤其适用于带哈希指纹的静态资源（如`app.a3bc4d.js`）  
- 避免浏览器因用户刷新触发的多余验证（常规缓存策略可能降级验证）  

---

## 解决方案  

### 配置示例（Nginx）  

```nginx
location /static/ {
    add_header Cache-Control "public, max-age=31536000, immutable";
    # 版本化资源设置长期缓存+immutable
    # 非版本化资源应避免使用，防止更新失效
}
```

### 优化建议  

1. **版本分离**：对内容哈希资源使用immutable，动态资源采用较短max-age  
2. **兜底机制**：HTML文件禁用强缓存，确保核心资源更新  
3. **监控覆盖**：通过Navigation Timing API统计缓存命中率  

---

## 深度追问  

1. **如何解决ETag在分布式系统的同步问题？**  

- 提示：使用一致性哈希算法或统一存储ETag生成规则  

2. **用户强制刷新（Ctrl+F5）时的缓存策略变化？**- 提示：请求头添加`Cache-Control: no-cache`绕过强缓存  

3. **如何验证缓存策略是否生效？**  

- 提示：浏览器DevTools网络面板查看请求状态码与响应头
