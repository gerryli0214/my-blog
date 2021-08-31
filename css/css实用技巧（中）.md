# CSS实用技巧（中）

## 前言

今年上半年，陆陆续续参加十来场前端招聘，发现很多前端`er`对`CSS`了解不是很多，如`vertical-align`、`BFC`、`position`等。本文就对面试中经常问到的易错知识点进行概括总结。从本文中，你将了解到以下内容：

- `vertical-align`生效条件及影响因素
- `BFC`是什么？有何作用？
- `position`的奇淫技巧

## CSS特性

### vertical-align为什么时灵时不灵

生效条件：只能应用在`display`为`inline`、`inline-block`、`inline-table`、`table-cell`上。

有个高频面试题，“如何使一个不定宽高`div`垂直水平居中？”，有的萌新竟然回答用`vertical-align: middle`。这个回答是减分的，至少在某种程度上给人一种感觉`CSS`基础比较薄弱。

#### 影响vertial-align外在表现的属性

- `line-height`

### BFC究竟有什么作用

#### 什么是BFC

#### BFC的作用

#### 什么是IFC

#### 延伸：如何清除浮动？

### position定位还能玩出什么花样

#### 普通position定位

#### left/top/right/bottom都有值的定位

#### left/top/right/bottom都没有值的定位
