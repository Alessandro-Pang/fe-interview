---
weight: 3033000
date: '2025-03-04T06:58:24.484Z'
draft: false
author: zi.Yang
title: 字符串截取方法区别
icon: javascript
toc: true
description: 请通过参数说明和示例演示substring()与substr()方法的核心差异，并指出为什么MDN文档建议开发者避免使用substr()方法？
tags:
  - javascript
  - 字符串
  - API设计
---

## 考察点分析

该题目考察候选人对字符串处理方法的掌握程度及代码规范意识，主要评估：

1. **API差异辨析能力**：准确区分易混淆的字符串方法参数定义与行为差异
2. **边界条件处理**：负数参数、反向索引、超长参数等特殊场景的处理逻辑
3. **标准规范认知**：理解ECMAScript标准演进及现代开发最佳实践

核心评估点：

- 参数定义差异（startIndex vs length）
- 负数参数处理逻辑
- 方法弃用背景与替代方案

## 技术解析

### 关键知识点

1. **参数定义**：
   - `substring(startIndex, endIndex)`：提取字符到结束索引前（左闭右开）
   - `substr(startIndex, length)`：从起始索引提取指定长度字符

2. **负数处理**：
   - `substring`：所有负数参数视为0
   - `substr`：start可为负（从末尾倒数），length为负时返回空

3. **标准化问题**：
   - `substr`非ECMA-262标准核心特性
   - MDN标注为遗留方法（Legacy feature）

### 原理剖析

```javascript
// 示例字符串
const str = "HelloWorld";

// substring(3,7) => "loWo"（索引3到6）
str.substring(3,7) 

// substr(3,7) => "loWorld"（从3开始取7字符）
str.substr(3,7) 

// 负数参数处理对比
str.substring(-5,3) => substring(0,3)="Hel"
str.substr(-5,3) => 从索引5开始取3字符 => "rld"
```

### 常见误区

- 混淆参数类型（将substr第二个参数误认为结束索引）
- 误判负数处理逻辑（特别是substr的start负索引）
- 忽视标准化差异（误以为substr是现代标准方法）

## 问题解答

`substring`与`substr`核心差异体现在参数定义与负数处理：

1. **参数语义**：
   - `substring(start, end)`：提取[start, end)区间字符
   - `substr(start, length)`：从start开始提取length个字符

2. **负数逻辑**：
   - `substring`将负数参数转换为0
   - `substr`的start负数为倒数，length负数返回空

MDN建议避免使用`substr`的原因：

- 非ECMAScript核心标准方法
- 存在跨浏览器兼容问题（如旧版IE的负数参数处理差异）
- 现代标准推荐使用`substring`或`slice`（符合直觉的负数处理）

## 解决方案

### 编码示例

```javascript
// 安全截取最后N位字符
function getLastNChars(str, n) {
  // 使用slice处理负数更直观
  return str.slice(-n); 
}

// 边界处理示例
console.log(getLastNChars("Hello", 3)); // "llo"
console.log(getLastNChars("Hi", 5)); // "Hi"（自动截断）
```

### 可扩展性建议

- 大流量场景优先使用`slice`（性能与`substring`相当）
- 低端设备避免链式操作（如`str.slice().trim()`分开处理）

## 深度追问

1. **如何用slice实现substr功能？**
   `str.substr(start, length)`等效于`str.slice(start, start+length)`

2. **处理含多字节字符时需要注意什么？**
   使用`Array.from(str).slice()`处理unicode字符

3. **性能敏感时如何优化字符串操作？**
   优先使用`slice`+预计算索引，避免链式操作
