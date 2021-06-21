# CSS实用技巧（上）

## 前言

张鑫旭的《CSS世界》这本书，强烈推荐前端er仔细阅读下，里面详细说明了许多不怎么被注意的`CSS`特性，对前端进阶很有帮助。
本文简要列举书中前四章比较实用的知识点，书中干货很多，值得一读。

## 实用技巧

### 文字少的时候居中显示，多的时候居左显示

利用元素的包裹性，元素的尺寸由内部元素决定，且永远小于容器的宽度。

**具有包裹性的元素：**`inline-block`、浮动元素、绝对定位元素。

```html
<style>
  .content{
    display: inline-block;
    text-align: left;
  }
  .box{
    text-align: center;
  }
</style>
<div class="box">
  <span class="content"></span>
</div>
```

### 你不知道的min-width/min-height, max-width/max-height

- 初始值

`min-width/min-height`初始值是`auto`,`max-height/max-width`初始值是`none`。

设置`min-height`渐变效果，需要指定`min-height`的值。

```html
<style>
  .box{
    min-height: 20px;
    width: 200px;
    background-color: coral;
    transition: min-height .5s;;
  }
  .box:hover{
    min-height: 300px;
  }
</style>
<template>
  <div class="box"></div>
</template>
```

- 覆盖规则

**超越important，超越最大**。简而言之，`min-width/min-height,max-width/max-height`会覆盖`width/height`。当`min-xxx`与`max-xxx`设置相冲突时，实际结果以`min-xxx`为准。

### 不可忽略的幽灵空白节点

如下代码，`div`没有设置宽高，`span`为空白标签，但是`div`的高度却为`18px`，这个高度是由字体大小和行高决定的，要想去除这个影响，需要将`font-size`设置为`0`。

```html
<style>
  div {
    background-color: #cd0000;
  }
  span {
    display: inline-block;
  }
</style>
<template>
 <div><span></span></div>
</template>
```

### 图片根据宽高自适应

指定图片宽高，使图片自适应宽高，常用的两种方式，第一种是`background`，第二种是`object-fit`。常用属性如下：

| background-size | object-fit | CSS属性 | 说明 |
| --- | --- | --- | --- |
| cover | cover | 覆盖 | 会存在图片展示不全 |
| contain | contain | 包含 | 等比缩放，空白自动填充 |
| -- | fill（默认） | 填充 | 不符合尺寸，横向、纵向压缩 |

### 功能强大的content

`CSS content`属性结合`before/after`伪类，能实现很多意想不到的效果。

- ...动态加载效果

```html
<style>
dot {
  display: inline-block;
  height: 1em;
  line-height: 1;
  text-align: left;
  vertical-align: -.25em;
  overflow: hidden;
} 
dot::before {
  display: block;
  content: '...\A..\A.';
  white-space: pre-wrap;
  animation: dot 3s infinite step-start both;
} 
@keyframes dot {
  33% { transform: translateY(-2em); }
  66% { transform: translateY(-1em); }
}
</style>
<template>
  <div>
    正在加载中<dot>...</dot>
  </div>
</template>
```

- `contenteditable`可编辑元素`placeholder`

```html
<style>
.edit{
  width: 200px;
  height: 50px;
  background-color: azure;
}
.edit:empty::before{
  content: attr(data-placeholder);
}
</style>
<template>
  <div class="edit" contenteditable="true" data-placeholder="this is a placeholder"></div>
</template>
```

### padding的妙用

`background-clip`可以设置`background`作用范围，结合`padding`，可以做出一些好玩的东西。

- 双层圆点

```html
<style>
.icon-dot {
  display: inline-block;
  width: 100px; height: 100px;
  padding: 10px;
  border: 10px solid;
  border-radius: 50%;
  background-color: currentColor;
  background-clip: content-box;
}
</style>
<template>
  <span class="icon-dot"></span>
</template>
```
  
### margin的奇淫技巧

- 左右两列等高布局

```html
<style>
.column-box {
  overflow: hidden;
} 
.column-left,
.column-right {
  margin-bottom: -9999px;
  padding-bottom: 9999px;
  float: left;
}
.column-left{
  width: 100px;
  background-color: #ccc;
}
.column-right{
  width: 100px;
  background-color: aquamarine;
}
</style>
<template>
<div class="column-box">
    <div class="column-left">
      123
    </div>
    <div class="column-right">
      456    
    </div>
  </div>
</template>
```

- 一端固定，另外一端自适应

```html
<style>
.left{
  width: 200px;
  height: 100%;
  float: left;
}
.right{
  margin-left: 200px;
}
</style>
<template>
  <body>
    <div class='left'></div>
    <div class='right'></div>
  </body>
</template>
```

- 块级元素垂直水平居中

```html
<style>
.father{
  width: 300px;
  height: 150px;
  position: relative;
}
.son{
  position: absolute;
  left: 0;
  right: 0;
  top: 0;
  bottom: 0;
  margin: auto;
}
</style>
<template>
  <div class='father'>
    <div class='son'></div>
  </div>
</template>
```
