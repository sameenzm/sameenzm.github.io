---
title: 总结React生命周期
date: 2018-07-06 14:55:51
tags: react
---

学习React，生命周期是必须知晓的。写了这么久React，一直没好好弄清楚生命周期的适用情况，今天归纳总结一下。

<!--more -->
### react生命周期介绍
开始：
constructor | 初始化

react生命周期分三个阶段:
实例化 | 挂载阶段
存在期 | 更新阶段
销毁期 | 卸载阶段


顺序 | 生命周期 | 阶段 | 说明&注意
---|---|---|---
1 | componentWillMount() | 挂载阶段 | (B/S), 如果在这里调用setState，本次的render函数可以看到更新后的state，并且只渲染一次。
2 | render | 挂载阶段 | 不要在render里面修改state
3 | √ componentDidMount() | 挂载阶段 | (B), 可以在这里使用refs,一般列表数据请求在这里
4 | componentWillReceiveProps(nextProps) | 更新阶段 | 在 props改变 时 被调用
5 | √ shouldComponentUpdate(nextProps, nextState) | 更新阶段 | props或state改变时调用，决定是否重新render组件，默认return true; 在比较复杂的应用里，有一些数据的改变并不影响界面展示，可以在这里做判断，优化渲染效率
6 | componentWillUpdate(nextProps, nextState) | 更新阶段 | props或state改变时调用，shouldComponentUpdate返回true或者调用forceUpdate之后，componentWillUpdate会被调用。
7 | render | 更新阶段 | 重新渲染啦~
8 | componentDidUpdate() | 更新阶段 | 除了首次render之后调用componentDidMount，其它render结束之后都是调用componentDidUpdate。这个方法在更新真实的DOM成功后调用，当我们需要访问真实的DOM时，这个方法就也经常用到。
9 | √ componentWillUnmount() | 卸载阶段 | 一般在componentDidMount里面注册的事件需要在这里删除。


比较常用的是 3 componentDidMount（首屏数据请求）、5 shouldComponentUpdate（性能优化）、9 componentWillUnmount（卸载数据、事件等）

### 生命周期更新相关
2018年3月29日, React 发布了16.3.0, 主要更新了两个点：新的生命周期函数和context API [React v16.3.0: New lifecycles and context API
](https://reactjs.org/blog/2018/03/29/react-v-16-3.html).

对于生命周期函数，主要有以下改变（[Update on Async Rendering](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)）
以下生命周期函数将在以后的版本（17.0）中弃用（加上UNSAFE_前缀）：
1 componentWillMount
4 componentWillReceiveProps
8 componentWillUpdate

添加了两个生命周期函数：
getDerivedStateFromProps
getSnapshotBeforeUpdate



[static getDerivedStateFromProps(nextProps, prevState)](https://reactjs.org/docs/react-component.html#static-getderivedstatefromprops)
 - 替代componentWillReceiveProps这个生命周期更加安全的方法
 - 在组件被实例化或接收到新的 props 时被调用;
 - 函数返回一个对象用于更新 state 以响应 props 变化。

[getSnapshotBeforeUpdate(prevProps, prevState)](https://link.zhihu.com/?target=https%3A//doc.react-china.org/docs/react-component.html%23getsnapshotbeforeupdate)
  - 为了支持在组件更新前更加安全的读取属
  - 这个函数会在最新的渲染输出提交给DOM前将会立即调用
  - 函数会返回一个Snapshot实例。在componentDidUpdate 函数中它可以作为第三个参数被调用，以便对DOM进行状态更新。





推荐阅读：
[React 新生命周期函数和迁移路径](https://zhuanlan.zhihu.com/p/37169569)