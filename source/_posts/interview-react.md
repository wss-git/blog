---
title: react 面试题
categories:
  - 面经
excerpt: ''
description: ''
date: 2023-12-12 15:19:54
---


## React 生命周期的各个阶段是什么？

constructor
static getDerivedStateFromProps 本身是一个静态方法，内部不能访问 this（componentWillMount 废弃）
render
componentDidMount

static getDerivedStateFromProps （ componentWillReceiveProps 废弃 钩子函数会在属性更新的时候执行 ）
shouldComponentUpdate 在这个钩子函数中可以通过 this 上当前的属性和状态与函数参数中接收到更改后的属性和状态进行对比，判断是否要继续执行其他的生命周期钩子函数
render
static getSnapshotBeforeUpdate 在这里可以提前进行一些数据的计算，将结果返回到 componentDidUpdate 中，进行逻辑分离， 之前在 componentDidUpdate 前会执行一个 componentWillUpdate

componentWillUnmount 卸载前

## React-hooks 有哪些 ,React 中提供了很多 Hooks，常用的有如下几个：

1、useState 用来创建持久化的状态
2、useRef 可以创建一个 ref 对象， 可以用于标记 dom 和子组件，也可以创建一个不会变化的持久数据
3、useEffect 可以用来监听数据变化以及模拟生命周期函数
4、useContext 可以高效便捷的取用 Context 中传递的 value

8、useReducer 可以创建一个小型的 store， 往往需要搭配 Context 使用
6、useMemo 实现 vue 中计算属性的能力，缓存计算结果， 提高性能
5、useCallback 用于缓存函数，避免函数重复构建， 提高性能
7、useLayoutEffect，与 useEffect 的区别是在 render 前同步执行
9、useImperativeHandle 可以给 React.forward 处理后的函数组件接收到的 ref 对象拓展功能
10、useid 可以创建一个唯一 id

## 用 hooks 编程的话如何取模拟生命周期

函数组件中利用 hooks 模拟生命周期主要使用的就是 useEffect
如果 useEffect 的依赖数组为空数组， 此时，模拟的是 componentDidMount
如果 useEffect 的依赖数组不为空，此时， 模拟的是 componnetDidUpdate
如果 useEffect 的回调函数中返回一个函数， 此时，这个返回的函数模拟的是 componentWillUnmount


## React 组件间传值的方法有哪些？

父组件通过 props 传递给子组件；
父组件通过 props 向子组件传入一个方法，子组件在通过调用该方法，将数据以参数的形式传给父组件，父组件可以在该方法中对传入的数据进行处理；
路由传值 props.params

## React 事件绑定原理

React 并不是将 click 事件绑在该 div 的真实 DOM 上
而是在 document 处监听所有支持的事件，当事件发生并冒泡至 document 处时，React 将事件内容封装并交由真正的处理函数运行
这样的方式不仅减少了内存消耗，还能在组件挂载销毁时统一订阅和移除事件
另外冒泡到 document 上的事件也不是原生浏览器事件，而是 React 自己实现的合成事件( SyntheticEvent )
因此如果不想要事件冒泡的话，调用 event.stopPropagation 是无效的，而应该调用 event.preventDefault

