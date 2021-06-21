# Fullscreen API与DOM监听API

## 前言

以下几个`API`，在`web`开发中可以简化我们一部分交互操作。

## Fullscreen API

有时候我们想要全屏预览的效果，比如类似于**图片预览、幻灯片播放**等。全屏`API`是一个很好的选择。

### 基本用法

- 打开全屏

```javascript
   element.requestFullscreen().then(() => {
       // 成功的处理
   }).catch(err => {
       // 失败的处理
   })
```

- 退出全屏

```javascript
   element.exitFullscreen().then(() => {
       // 成功的处理
   }).catch(err => {
       // 失败的处理
   })
```

- 事件

`fullscreenchange`监听元素全屏、退出全屏的变化；`fullscreenerror`监听全屏请求失败。

### 注意事项

在`iframe`中，如果要手动调用`fullscreen API`，可能会遇到错误`Document is not active`，是因为没有给`iframe`元素赋予`fullscreen`的权限。此时应该给`iframe`增加属性`allowfullscreen`。

## MutationObserver

`MutationObserver`目前是`DOM`变化监听神器。主要原因有二：

1. 异步监听`DOM`变化，不影响页面性能
2. 可以细粒度的监听`DOM`内容变化，元素、属性、文本的变化都能检测到，且能够比较出新旧值的差异

在一定程度上，使用它能够做到文档内容的撤销、重做。

### 基本用法

- 实例化

```javascript

var observerInstance = new MutationObserver(function (mutations, observer) {
    // mutations为变化内容描述
    mutations.forEach(function(mutation) {
        console.log(mutation);
    });
});

```  

每次`DOM`变化，都会触发实例化中的回调方法

- 方法

1. 添加元素监听

```javascript
// 监听body中元素的变化
observerInstance.observer(document.body, {
    attributes: true, // 属性
    characterData: true, // 文本
    childList: true, // 子节点
    subtree: true, // 后代节点
    attributeOldValue: true, // 属性旧值
    characterDataOldValue: true // 旧文本
})
```

2. 移除元素监听

```javascript
observerInstance.disconnect()
```

调用此方法后，将停止对已添加元素的监听

3. 清楚变动记录

```javascript
observerInstance.takeRecords()
```

因为监听是异步的，不是每次`DOM`变动都实时触发，此方法的调用，会使历史缓存的变化信息被清空。

## IntersectionObserver

`IntersectionObserver`与`MutationObserver`使用方法比较类似，原理也有点类似，只是前者的主要作用是监测元素的**可见性变化**，后者主要是**监测`DOM`的变化**。两者都是异步操作，不会影响页面性能。
利用此`API`，实现上拉列表刷新，下拉加载更多将会变得非常简单。

### 基本用法

- 实例化

```javascript

var observerInstance = new IntersectionObserver(function (entries) {
    // intersectionRatio代表元素的可见比例，可见大于0，不可见小于等于0
    if (entries[0].intersectionRatio <= 0) return;

    loadItems(10);
}, {
    threshold: [0, 0.25, 0.5, 0.75, 1] // 监听对象的交叉区域与边界区域的比率,默认是0
  });

```  

- 方法

1. 添加元素监听

```javascript
// 监听特定元素，每次调用都会重新添加一个元素的监听
observerInstance.observer(document.getElement('item1'))
```

2. 移除元素监听

```javascript
observerInstance.disconnect()
```

调用此方法后，将停止对已添加元素的监听

3. 返回所有观察目标数组

```javascript
let IntersectionObserverEntryList = observerInstance.takeRecords()
```

4. 停止观察特定元素

```javascript
observerInstance.unobserver()
```

## 参考文件

- [Fullscreen API](https://developer.mozilla.org/zh-CN/docs/Web/API/Fullscreen_API/%E6%8C%87%E5%8D%97)
- [MutationObserver](https://javascript.ruanyifeng.com/dom/mutationobserver.html)
- [IntersectionObserver](https://developer.mozilla.org/zh-CN/docs/Web/API/IntersectionObserver)
