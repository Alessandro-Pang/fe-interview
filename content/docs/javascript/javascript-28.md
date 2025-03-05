---
weight: 3800
date: '2025-03-04T06:58:24.483Z'
draft: false
author: zi.Yang
title: JSON与对象转换规范
icon: javascript
toc: true
description: >-
  请说明JSON.stringify()方法在序列化JavaScript对象时的特殊处理规则，包括对undefined、函数和循环引用的处理方式，并解释为什么JSON不是JavaScript对象的严格子集？
tags:
  - javascript
  - JSON
  - 数据格式
---

## 考察点分析

**核心能力维度**：JavaScript核心API理解、数据序列化规范、异常处理能力  

1. **特殊值处理机制**：对undefined、函数等非标准JSON类型的处理规则  
2. **循环引用检测**：JSON.stringify的异常处理机制  
3. **JSON语法规范**：与JavaScript对象字面量的差异分析  
4. **边界场景处理**：嵌套对象、数组等不同数据结构的序列化表现  
5. **规范差异理解**：JSON与JavaScript对象的语法兼容性问题  

---

## 技术解析

### 关键知识点优先级

JSON序列化规范 > 循环引用检测 > 数据类型兼容性 > 语法差异

### 原理剖析

1. **特殊值处理**：
   - `undefined`、函数、Symbol类型在被序列化为**对象属性值**时会被忽略（对象结构）
   - 作为**数组元素**时：`undefined`转为`null`，函数/Symbol转为`null`（数组结构）
   - `Date`对象自动调用`toJSON()`转为ISO字符串

2. **循环引用检测**：

   ```javascript
   const obj = { a: 1 };
   obj.self = obj;  // 循环引用
   JSON.stringify(obj); // 抛出TypeError
   ```

   序列化过程使用递归遍历对象树，通过内部缓存检测循环引用

3. **JSON非严格子集原因**：
   - **语法差异**：JSON要求字符串键名必须双引号，不允许尾随逗号
   - **字符转义**：JSON严格规定`\u2028`/`\u2029`必须转义（JS字符串允许直接存在）
   - **数值格式**：JSON不支持`NaN`/`Infinity`，需转为`null`

### 常见误区

1. 误认为数组中的`undefined`会被忽略（实际转为`null`）
2. 期望`JSON.parse()`能还原函数（实际仅处理标准JSON类型）
3. 忽略循环引用场景的异常处理导致程序崩溃

---

## 问题解答

JSON.stringify()在序列化时：  

1. **特殊值处理**：对象属性中的`undefined`/函数/Symbol被忽略，数组项中转为`null`；
2. **循环引用**：检测到循环引用时抛出TypeError；  
3. **JSON非子集**：因语法规范差异（引号要求/特殊字符转义）和数据类型限制（缺少JS特有类型支持），合法JSON可能在JS中无法直接解析为有效对象。

---

## 解决方案

### 循环引用处理示例

```javascript
function safeStringify(obj) {
  const seen = new WeakSet();
  
  return JSON.stringify(obj, (key, value) => {
    if (typeof value === 'object' && value !== null) {
      if (seen.has(value)) return '[Circular]';
      seen.add(value);
    }
    // 处理其他特殊类型
    return value instanceof Symbol ? value.toString() : value;
  }, 2);
}

const obj = { a: undefined, arr: [() => {}] };
obj.self = obj;
console.log(safeStringify(obj)); 
// 输出：{"arr":[null],"self":"[Circular]"}
```

**优化说明**：  

- 使用WeakSet实现O(1)时间复杂度的引用检测
- 空间复杂度控制在O(n)（n为对象树节点数）

**扩展建议**：  

- 大数据量时使用流式处理替代全内存操作
- 低端设备可启用缓存检测阈值

---

## 深度追问

1. **如何让JSON.parse()处理Date字符串自动转为Date对象？**  
   *提示：使用reviver函数检测日期格式并转换*

2. **JSON规范为何要排除undefined和函数？**  
   *提示：数据交换需要语言无关的通用格式*

3. **哪些JSON合法结构在JS中会引发语法错误？**  
   *提示：单引号键名、十六进制数字字面量*
