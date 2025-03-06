---
weight: 11004000
date: '2025-03-04T09:31:00.136Z'
draft: false
author: zi.Yang
title: 事件循环与任务队列管理
icon: public
toc: true
description: >-
  结合宏任务（macrotask）与微任务（microtask）的执行顺序，解析浏览器事件循环如何调度setTimeout、Promise、MutationObserver等异步任务。说明渲染帧（requestAnimationFrame）与事件循环的协作关系。
tags:
  - network
  - 事件循环
  - 异步机制
  - 任务调度
---

## 考察点分析

本题主要考查以下核心能力维度：

1. **事件循环机制**：理解浏览器Event Loop的工作流程及任务队列管理
2. **异步任务调度**：区分宏任务与微任务的执行优先级及嵌套处理
3. **渲染管线整合**：掌握requestAnimationFrame与浏览器渲染流程的协作关系

具体技术评估点包括：

- 宏任务(macrotask)与微任务(microtask)的定义与执行顺序
- 微任务队列的清空机制及嵌套处理
- requestAnimationFrame在渲染帧中的触发时机
- 浏览器渲染流程与事件循环的协作关系

---

## 技术解析

### 关键知识点

Event Loop > 微任务队列 > 宏任务队列 > requestAnimationFrame > 渲染管线

### 原理剖析

浏览器事件循环的完整流程：

1. **执行宏任务**：从宏任务队列取出最老任务（如script整体代码、setTimeout回调）
2. **清空微任务队列**：执行所有微任务（Promise.then、MutationObserver），若微任务中产生新微任务，会持续执行直至队列清空
3. **渲染前阶段**：执行requestAnimationFrame回调
4. **渲染流程**：样式计算 → 布局 → 绘制
5. **宏任务循环**：重复上述过程

### 常见误区

1. 认为微任务在宏任务之后执行（实际在**当前宏任务结束后立即执行**）
2. 忽略微任务队列需要完全清空的特性（嵌套微任务会阻塞渲染）
3. 混淆requestAnimationFrame与宏/微任务的执行阶段

---

## 问题解答

浏览器事件循环按阶段调度任务：

1. **宏任务阶段**：执行一个脚本或setTimeout回调等任务
2. **微任务阶段**：立即执行该宏任务产生的所有微任务，包括嵌套微任务
3. **渲染准备阶段**：执行requestAnimationFrame回调
4. **渲染阶段**：处理样式计算、布局与绘制
5. **循环触发**：从宏任务队列取下一个任务

示例代码解析：

```javascript
console.log('Start');

setTimeout(() => console.log('Timeout')); // 宏任务

Promise.resolve().then(() => {
  console.log('Promise');
  Promise.resolve().then(() => console.log('Nested Promise')); // 嵌套微任务
});

requestAnimationFrame(() => console.log('RAF'));

console.log('End');
```

输出顺序：Start → End → Promise → Nested Promise → RAF → Timeout

---

## 解决方案

### 编码示例

```javascript
function scheduleTasks() {
  // 宏任务1：主线程代码
  console.log('宏任务1开始');
  
  setTimeout(() => { // 宏任务2
    console.log('setTimeout回调');
  });

  Promise.resolve().then(() => { // 微任务1
    console.log('Promise1');
    Promise.resolve().then(() => console.log('嵌套微任务')); 
  });

  requestAnimationFrame(() => console.log('RAF回调')); 

  console.log('宏任务1结束');
}

// 输出顺序：
// 宏任务1开始 → 宏任务1结束 → Promise1 → 嵌套微任务 → RAF回调 → setTimeout回调
```

### 可扩展性建议

1. **高频动画场景**：用requestAnimationFrame替代setTimeout保证帧同步
2. **批量DOM操作**：在微任务中处理DOM变更，通过MutationObserver监听
3. **长任务优化**：将耗时操作拆解为多个微任务，避免阻塞渲染

---

## 深度追问

1. **如何监控微任务堆积？**
   提示：使用PerformanceObserver监听事件循环延迟

2. **requestAnimationFrame与CSS动画的执行顺序？**
   提示：RAF在样式计算前执行，CSS动画在渲染管线中处理

3. **Node.js与浏览器的事件循环差异？**
   提示：Node.js有额外阶段（如poll/check阶段）
