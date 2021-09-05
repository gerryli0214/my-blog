# CSS实用技巧（中）

## 前言

今年上半年，陆陆续续参加十来场前端招聘，发现很多前端`er`对`CSS`了解不是很多，如`vertical-align`、`BFC`、`position`等。本文就对面试中经常问到的易错知识点进行概括总结。从本文中，你将了解到以下内容：

- `vertical-align`为何时灵时不灵
- `BFC`是什么？有何作用
- `position`的奇淫技巧

## CSS特性

### vertical-align为什么时灵时不灵

#### 生效条件

只能应用在`display`为`inline`、`inline-block`、`inline-table`、`table-cell`上。

有个高频面试题，“如何使一个不定宽高`div`垂直水平居中？”，有的萌新竟然回答用`vertical-align: middle`。这个回答是减分的，至少在某种程度上给人一种感觉`CSS`基础比较薄弱。

#### 内联元素垂直居中对齐

开发中会遇到用字幕`x`代替关闭`icon`，用`...`显示溢出或者加载中。但是会发现字母`x`、省略号并没有与文本垂直方向居中对齐，这是因为文本默认是基线对齐，`x`、省略号默认底部在基线处。如下图所示：

![x-height](../asserts/x-height.png)

如下，为文本对齐`demo`:

```html
<div class="container">
  <span>你好，世界</span>
  <span class="more">...</span>
</div>
```

实际显示效果如下：

![baseline](../asserts/baseline.png)

如果要实现垂直居中，利用`vertical-align`，搭配`line-height`即可，`vertical-align`不仅可以设置`middle/top/bottom/baseline...`关键字，也可以设置常用的度量单位，正负值均可，使用比较灵活。为什么要给`.more`设置`line-height`属性呢？其实是因为`line-height`属性可以继承，如果不缩小`.more`的行高，就会撑大父元素的尺寸。

```html
<style>
  .container{
    font-size: 64px;
    line-height: 64px;
  }
  .more{
    line-height: 16px;
    vertical-align: 16px;
  }
</style>
```

### BFC究竟有什么作用

#### 什么是BFC

`BFC`全称`block formatting context`，即“块状格式化上下文”，与外界元素相对独立的一片区域，具有以下特性：

- 计算`BFC`高度时，浮动元素也参与计算
- 属于同一`BFC`容器的元素垂直方向的`margin`会合并
- `BFC`容器是独立容器，不会影响外部元素的布局

利用`BFC`的特性，我们可以实现以下功能：

1. 清除浮动
2. 防止垂直方向`margin`合并
3. 实现多栏弹性布局

#### BFC的生效条件

以下`CSS`属性会触发元素生成`BFC`结界:

- 根元素（`<html>`）
- 浮动元素（元素的 `float` 不是 `none`）
- 绝对定位元素（元素的 `position` 为 `absolute` 或 `fixed`）
- 行内块元素（元素的 `display` 为 `inline-block`）
- 表格单元格（元素的 `display` 为 `table-cell`，`HTML`表格单元格默认为该值）
- 表格标题（元素的 `display` 为 `table-caption`，`HTML`表格标题默认为该值）
- 匿名表格单元格元素（元素的 `display` 为 `table`、`table-row`、 `table-row-group`、`table-header-group`、`table-footer-group`（分别- 是`HTML table`、`row`、`tbody`、`thead`、`tfoot` 的默认属性）或 `inline-table`）
- `overflow` 计算值(`Computed`)不为 `visible` 的块元素
- `display` 值为 `flow-root` 的元素
- `contain` 值为 `layout`、`content` 或 `paint` 的元素
- 弹性元素（`display` 为 `flex` 或 `inline-flex` 元素的直接子元素）
- 网格元素（`display` 为 `grid` 或 `inline-grid` 元素的直接子元素）
- 多列容器（元素的 `column-count` 或 `column-width` 不为 `auto`，包括 `column-count` 为 1）
- `column-span` 为 `all` 的元素始终会创建一个新的`BFc`

#### BFC使用案例

- 清除浮动

```html
<style>
  .container{
    /* overflow: hidden; */
    /* position: absolute; */
    /* float: left; */
  }
  .left{
    float: left;
    width: 200px;
    height: 200px;
  }
</style>
<div class="container">
  <div class="left"></div>
</div>
```

以上代码，`container`容器高度为`0`，因为子元素`left`浮动。我们只需要把`container`容器转成`BFC`容器，即可清楚浮动，注释的几种方法都可以。

- 防止垂直方向`margin`合并

```html
<style>
  .blue, .red-inner {
    height: 50px;
    margin: 10px 0;
  }

  .blue {
    background: blue;
  }

  .red-outer {
    overflow: hidden;
    background: red;
  }
</style>
<div class="blue"></div>
<div class="red-outer">
  <div class="red-inner">red inner</div>
</div>
```

- 自适应布局

左侧固定，右侧自适应。

```html
<style>
  .left{
    height: 200px;
    width: 200px;
    float: left;
    background-color: burlywood;
  }
  .right{
    height: 200px;
    margin-left: 200px;
    background-color: cadetblue;
  }
</style>
<div class="container">
  <div class="left"></div>
  <div class="right"></div>
</div>
```

### position定位还能玩出什么花样

#### 普通position定位

#### left/top/right/bottom都有值的定位

#### left/top/right/bottom都没有值的定位

## 参考资料

- [BFC](https://developer.mozilla.org/zh-CN/docs/Web/Guide/CSS/Block_formatting_context)
