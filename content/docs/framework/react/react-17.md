---
weight: 6017000
date: '2025-03-05T12:28:17.276Z'
draft: false
author: zi.Yang
title: 避免不必要的渲染策略
icon: icon/react.svg
toc: true
description: >-
  列举3种避免React组件不必要渲染的方法，如类组件中的`shouldComponentUpdate`、函数组件中的`React.memo`，并解释其底层浅比较的原理与限制？
tags:
  - react
  - 性能优化
  - 渲染控制
  - 浅比较
---

## 考察点分析

本题主要考察以下核心能力维度：

1. **React性能优化理解**：评估对React渲染机制及优化策略的掌握程度
2. **组件更新机制原理**：检验对虚拟DOM、组件生命周期、渲染触发条件的理解深度
3. **不可变数据实践**：考察对引用类型数据处理的工程化思维

具体技术评估点：

- 类组件生命周期方法`shouldComponentUpdate`的实现与限制
- 函数组件`React.memo`的浅比较机制及高阶组件应用
- `PureComponent`与普通组件的行为差异
- 浅比较（Shallow Compare）的底层实现原理
- 引用类型数据在优化策略中的处理要点

---

## 技术解析

### 关键知识点

1. `shouldComponentUpdate` > `PureComponent` > 浅比较原理
2. `React.memo` > 高阶组件 > 引用稳定性
3. 不可变数据操作 > 性能优化策略

### 原理剖析

**浅比较工作机制**：

- 对基本类型值进行`Object.is`比较
- 对对象类型进行引用地址比对
- 仅比较props第一层属性（Shallow Compare）
- 时间复杂度O(n)，空间复杂度O(1)

**典型执行流程**：

```
触发更新 → 执行SCU（或自动浅比较）→ 对比新旧props/state → 
│  无变化 → 阻止渲染              
└  有变化 → 继续渲染流程
```

### 常见误区

1. 误认为浅比较会递归对比嵌套对象
2. 在`shouldComponentUpdate`中直接修改state导致死循环
3. 忽略函数类型prop的引用变化问题
4. 在渲染函数中动态创建对象导致优化失效

---

## 问题解答

在React中避免不必要渲染的三种策略：

1. **shouldComponentUpdate（类组件）**  
手动控制更新决策，通过对比nextProps/nextState与当前值的浅比较返回布尔值。典型实现：

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return !shallowEqual(this.props, nextProps) || 
         !shallowEqual(this.state, nextState)
}
```

2. **React.memo（函数组件）**  
高阶组件缓存渲染结果，对props进行浅比较。可自定义比较函数：

```javascript
const MemoizedComp = React.memo(MyComp, (prev, next) => {
  return shallowEqual(prev, next)
})
```

3. **PureComponent（类组件）**  
自动实现props和state的浅比较，相当于内置`shouldComponentUpdate`的优化版本。继承方式：

```javascript
class MyComp extends React.PureComponent {
  // 自动进行浅比较
}
```

**浅比较限制**：

- 无法检测嵌套对象变化（如`obj.value`改变但`obj`引用未变）
- 数组长度或元素顺序变化但引用未更新时失效
- 函数类型props的引用稳定性问题

---

## 解决方案

### 编码示例

```javascript
// 优化后的列表组件
const MemoizedList = React.memo:function List({ items }) {
  return items.map((item, index) => (
    <ListItem key={item.id} item={item} />
  ))
}, (prev, next) => {
  // 自定义比对逻辑
  return prev.items.length === next.items.length 
      && prev.items.every((item, i) => 
          shallowEqual(item, next.items[i]))
})

// 配合useMemo稳定数据引用
function ParentComp() {
  const items = useMemo(() => fetchData(), [deps])
  return <MemoizedList items={items} />
}
```

**优化策略**：

1. 对列表项使用稳定的key
2. 使用Immutable.js等库确保数据变更产生新引用
3. 对高频更新组件设置合理memoization

---

## 深度追问

1. **如何避免深层对象属性变更导致的更新失效？**  
采用不可变数据模式（如immer），或手动触发引用变更

2. **React.memo与useMemo的区别是什么？**  
memo缓存组件实例，useMemo缓存计算结果

3. **什么场景下应该避免使用这些优化？**  
组件本身渲染代价小时，优化成本可能高于收益
