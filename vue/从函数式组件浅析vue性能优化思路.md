# 从函数式组件浅析vue性能优化思路

## 简介

`vue`[函数式组件](https://cn.vuejs.org/v2/guide/render-function.html#%E5%87%BD%E6%95%B0%E5%BC%8F%E7%BB%84%E4%BB%B6)大部分人在开发过程中用到的不多，就连官方文档位置放置的也比较隐晦，但是在我们对项目做性能优化时，却是一个不错的选择。本文将对函数式组件初始化过程做一个系统性的阐述，通过本文，你将了解到以下内容：

- 什么是函数式组件
- 函数式组件实现原理
- `vue`性能优化思路

## 什么是函数式组件

函数式组件即**无状态组件**，没有`data`、`computed`、`watch`，也没有生命周期方法，组件中也没有`this`上下文，只有`props`传参。在开发中，有很多组件仅仅只用到了`props`和插槽，这部分组件就可以提炼为函数式组件。借用官网`demo`，最简单的函数式组件如下：

```JavaScript
Vue.component('my-component', {
  functional: true,
  // Props 是可选的
  props: {
    // ...
  },
  // 为了弥补缺少的实例
  // 提供第二个参数作为上下文
  render: function (createElement, context) {
    // ...
  }
})
```

## 函数式组件实现原理

组件实例化过程大致分为四步，状态初始化 --> 模板编译 --> 生成`VNode` --> 转换为真实`DOM`。接下来对比普通组件与函数式组件常用配置项，比较下差异。

| 功能点名称 | 普通组件 | 函数式组件 | 描述 |
| ---- | ---- | ---- | ---- |
| vm | Y | N | 组件作用域 |
| hooks | Y | N | 生命周期钩子 |
| data | Y | N | 数据对象声明 |
| computed | Y | N | 计算属性 |
| watch | Y | N | 侦听器 |
| props | Y | Y | 属性 |
| children | Y | Y | VNode 子节点的数组 |
| slots | Y | Y | 一个函数，返回了包含所有插槽的对象 |
| scopedSlots | Y | Y | 作用域插槽的对象 |
| injections | Y | Y | 依赖注入 |
| listeners | Y | Y | 事件监听 |
| parent | Y | Y | 对父组件的引用 |

从上表中可以看出，普通组件与函数式组件最大的差别在于函数式组件**没有独立作用域**，没有响应式数据声明。没有独立作用域，会有以下优点：

1. 没有组件实例化(`new vnode.componentOptions.Ctor(options)`)，函数式组件获取`VNode`仅仅是普通函数调用

   - 无公共属性、方法拷贝
   - 无生命周期钩子调用

2. 函数式组件直接挂载到父组件中，

## vue性能优化思路
