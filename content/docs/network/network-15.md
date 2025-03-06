---
weight: 11015000
date: '2025-03-04T09:31:00.145Z'
draft: false
author: zi.Yang
title: Cache-Control指令集详解
icon: public
toc: true
description: >-
  解析max-age、no-cache、no-store、must-revalidate等Cache-Control指令的优先级关系，并给出电商页面与静态资源站点的差异化缓存策略设计示例。
tags:
  - network
  - 缓存控制
  - HTTP头部
  - 策略设计
---

## 回答

### 一、考察点分析  

**核心能力维度**：HTTP缓存机制理解、缓存策略设计能力、业务场景分析能力  
**技术评估点**：  

1. Cache-Control指令优先级逻辑  
2. 浏览器与CDN的缓存行为差异  
3. 动态内容与静态资源的缓存特征  
4. 缓存验证机制（如ETag/Last-Modified）协同工作  
5. 不同业务场景下的TTL(Time To Live)设计原则  

---

### 二、技术解析  

**关键知识点**：缓存指令优先级 > 缓存存储规则 > 缓存验证流程  

**原理剖析**：  

1. **优先级规则**：  

   ```
   no-store(禁用缓存) > must-revalidate(强制验证 > no-cache(协商缓存) > max-age(强缓存)
   ```  

   当存在冲突指令时，限制性更强的指令生效。例如同时设置`no-store`和`max-age=3600`时，浏览器不会存储任何缓存副本。

2. **指令详解**：  
   - `max-age=N`：资源在N秒内被视为新鲜，优先使用本地缓存（强缓存）  
   - `no-cache`：每次需携带验证信息到服务端检查（协商缓存）  
   - `no-store`：禁止任何形式的缓存存储  
   - `must-revalidate`：过期缓存必须验证有效性，但新鲜期内可直接使用  

3. **常见误区**：  
   - 混淆`no-cache`与`no-store`：前者允许存储但需验证，后者完全禁止存储  
   - 误认为`max-age=0`等价于`no-cache`（实际需要配合`must-revalidate`）  
   - 忽略CDN对缓存指令的特殊处理（如部分CDN会忽略`no-store`）  

---

### 三、问题解答  

**指令优先级**：当响应头包含多个冲突指令时，按`no-store > must-revalidate > no-cache > max-age`顺序生效。例如同时设置`no-cache, max-age=3600`时，浏览器将执行协商缓存而非强缓存。

**场景策略设计**：  

1. 电商页面（动态内容）：  

   ```http
   Cache-Control: no-cache, max-age=0, must-revalidate
   ```  

   配合`ETag`实现秒级更新价格库存，避免用户看到过期数据。

2. 静态资源站点：  

   ```http
   Cache-Control: public, max-age=31536000, immutable
   ```  

   通过文件哈希实现永久缓存，配合CDN边缘节点加速，资源更新时修改URL指纹。

---

### 四、解决方案  

**编码示例（Nginx配置）**：  

```nginx
# 电商动态页面
location /product {
    add_header Cache-Control "no-cache, max-age=0, must-revalidate";
    add_header ETag $sent_http_etag;
}

# 静态资源
location /static {
    expires 1y;
    add_header Cache-Control "public, max-age=31536000, immutable";
}
```

**可扩展性建议**：  

1. 灰度发布场景：通过`Vary`头实现AB测试缓存隔离  
2. 大流量优化：静态资源设置`stale-while-revalidate`实现后台异步更新  
3. 弱网环境：动态内容使用`private`指令避免CDN缓存  

---

### 五、深度追问  

1. **如何应对缓存击穿问题**？  
   答：使用互斥锁+后台更新机制，或设置`stale-while-revalidate`

2. **ETag与Last-Modified的优先级关系**？  
   答：ETag优先，因其精度更高

3. **Service Worker如何影响缓存指令**？  
   答：SW可覆盖HTTP缓存策略，需注意版本控制
