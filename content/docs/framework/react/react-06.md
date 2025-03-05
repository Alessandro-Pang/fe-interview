---
weight: 1600
date: '2025-03-05T12:28:17.275Z'
draft: false
author: zi.Yang
title: React生命周期阶段与方法
icon: icon/react.svg
toc: true
description: >-
  列举React类组件的生命周期阶段（挂载、更新、卸载）及其对应方法（如`componentDidMount`、`shouldComponentUpdate`），并说明各方法的典型使用场景？
tags:
  - react
  - 生命周期
  - 类组件
  - 阶段方法
---

## 考察点分析

本题主要考察候选人以下能力维度：

1. **框架机制理解**：对React类组件生命周期的阶段性划分及执行顺序的掌握
2. **API运用经验**：辨别各生命周期方法的使用场景及废弃API的演进
3. **性能优化意识**：理解关键方法（如shouldComponentUpdate）在渲染优化中的作用

具体评估点：

- 挂载阶段各方法的执行顺序与职责划分
- 更新阶段的条件触发与优化控制
- 卸载阶段资源回收的必要性
- 废弃生命周期方法的替代方案认知
- 错误使用场景辨别（如setState使用限制）

---

## 技术解析

### 关键知识点

组件生命周期 > 阶段划分 > 方法执行顺序 > 使用场景 > 废弃API演进

### 原理剖析

React类组件通过生命周期方法实现"过程式编程"，核心流程分为三个阶段：

1. **挂载阶段**（Mounting）：
   - `constructor`：初始化状态、绑定方法
   - `static getDerivedStateFromProps(props, state)`：根据props派生state
   - `render`：生成虚拟DOM
   - `componentDidMount`：DOM就绪后执行副作用（API请求、DOM操作）

2. **更新阶段**（Updating）：
   - `static getDerivedStateFromProps`：props变化时更新state
   - `shouldComponentUpdate(nextProps, nextState)`：返回布尔值控制是否渲染
   - `render`：重新生成虚拟DOM
   - `getSnapshotBeforeUpdate( (prevProps, prevState)`：获取DOM快照
   - `componentDidUpdate( (prevProps, prevState, snapshot)`：更新后执行操作

3. **卸载阶段**（Unmounting）：
   - `componentWillUnmount`：清理定时器、取消订阅

### 常见误区

- 在`constructor`中直接复制props到state（应使用`getDerivedStateFromProps`）
- `componentDidUpdate`中无限制调用setState导致死循环
- 混淆`componentWillReceiveProps`与`getDerivedStateFromProps`的使用场景
- 在`render`中执行副作用操作破坏纯函数特性

---

## 问题解答

React类组件生命周期分为挂载、更新、卸载三个阶段：

**挂载阶段**：

1. `constructor`：初始化state与事件绑定
2. `getDerivedStateFromProps`（静态）：根据props派生state
3. `render`：生成虚拟DOM结构
4. `componentDidMount`：发起网络请求、DOM操作、事件订阅

**更新阶段**：

1. `getDerivedStateFromProps`：props更新时同步state
2. `shouldComponentUpdate`：通过浅比较阻止无效渲染（返回false时流程中断）
3. `render`：生成新虚拟DOM
4. `getSnapshotBeforeUpdate`：获取更新前DOM状态（如滚动位置）
5. `componentDidUpdate`：基于新DOM进行操作（需条件判断避免循环）

**卸载阶段**：
`componentWillUnmount`：清除定时器、取消网络请求、移除事件监听

典型场景示例：

- `componentDidMount`：数据初始化请求（`fetch('/api')`）
- `shouldComponentUpdate`：比较新旧props避免子组件无效重绘
- `componentDidUpdate`：表单验证后焦点管理
- `componentWillUnmount`：清除`setInterval`计时器

---

## 解决方案

### 编码示例

```javascript
class LifecycleDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  static getDerivedStateFromProps(props, state) {
    // 根据props派生state（谨慎使用）
    return props.initCount > 10 ? { count: 10 } : null;
  }

  componentDidMount() {
    this.timer = setInterval(() => {
      console.log('计时器运行中...');
    }, 1000);
    // 发起API请求
    fetch('/api/data').then(/* ... */);
  }

  shouldComponentUpdate(nextProps, nextState) {
    // 仅当count变化时重新渲染
    return nextState.count !== this.state.count;
  }

  getSnapshotBeforeUpdate(prevProps, prevState) {
    // 记录滚动位置
    return this.listRef.scrollHeight;
  }

  componentDidUpdate(prevProps, prevState, snapshot) {
    if (snapshot !== null) {
      this.listRef.scrollTop += this.listRef.scrollHeight - snapshot;
    }
    // 更新后校验
    if (this.state.count > 100) {
      this.setState({ count: 100 });
    }
  }

  componentWillUnmount() {
    clearInterval(this.timer); // 清除定时器
  }

  handleClick() {
    this.setState(prev => ({ count: prev.count + 1 }));
  }

  render() {
    return (
      <div ref={el => this.listRef = el}>
        <button onClick={this.handleClick}>点击计数</button>
        <div>{this.state.count}</div>
      </div>
    );
  }
}
```

### 可扩展性建议

1. **性能优化**：对复杂组件使用`React.PureComponent`自动浅比较
2. **错误边界**：配合`componentDidCatch`捕获子组件错误
3. **异步模式**：在`componentWillUnmount`中取消未完成的异步操作
4. **大型应用**：将副作用逻辑移入HOC或自定义Hook提升可维护性

---

## 深度追问

1. **如何避免`getDerivedStateFromProps`的常见陷阱？**
   - 优先考虑完全受控组件或使用`key`重置非受控状态

2. **函数组件中如何模拟生命周期？**
   - `useEffect`依赖数组实现不同生命周期阶段

3. **React 18并发模式下生命周期有何变化？**
   - 部分方法可能被多次调用，需保证幂等性
