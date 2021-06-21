# Vue插槽实现原理

## 前言

`vue.js`的灵魂是组件，而组件的灵魂是插槽。借助于插槽，我们能最大程度上实现组件复用。本文主要是对插槽的实现机制进行详细概括总结，在某些场景中，有一定的用处。知其然知其所以然，掌握`vue.js`实现原理，不仅可以提升自身解决问题的能力，还可以学习到大神们编程思想和开发范式。

## 样例代码

```html
<!-- 子组件comA -->
<template>
  <div class='demo'>
    <slot><slot>
    <slot name='test'></slot>
    <slot name='scopedSlots' test='demo'></slot>
  </div>
</template>
<!-- 父组件 -->
<comA>
  <span>这是默认插槽</span>
  <template slot='test'>这是具名插槽</template>
  <template slot='scopedSlots' slot-scope='scope'>这是作用域插槽（老版）{{scope.test}}</template>
  <template v-slot:scopedSlots='scopeProps' slot-scope='scope'>这是作用域插槽（新版）{{scopeProps.test}}</template>
</comA>

```
  
## 透过现象看本质

插槽的作用是**实现内容分发**，实现内容分发，需要两个条件：

1. 占位符
2. 分发内容

组件内部定义的`slot`标签，我们可以理解为**占位符**，父组件中插槽内容，就是要**分发的内容**。插槽处理本质就是将指定内容放到指定位置。废话不多说，从本篇文章中，将能了解到：

1. 插槽的实现原理
2. `render`方法中如何使用插槽

## 实现原理

`vue`组件实例化顺序为：父组件状态初始化(`data`、`computed`、`watch`...) --> 模板编译 --> 生成`render`方法 --> 实例化渲染`watcher` --> 调用`render`方法，生成`VNode` --> `patch VNode`，转换为真实`DOM` --> 实例化子组件 --> ......重复相同的流程 --> 子组件生成的真实`DOM`挂载到父组件生成的真实`DOM`上，挂载到页面中 --> 移除旧节点

从上述流程中，可以推测出：

1. 父组件模板解析在子组件之前，所以父组件首先会获取到插槽模板内容
2. 子组件模板解析在后，所以在子组件调用`render`方法生成`VNode`时，可以借助部分手段，拿到插槽的`VNode`节点
3. 作用域插槽可以获取子组件内变量，因此作用域插槽的`VNode`生成，是动态的，即需要实时传入子组件的作用域`scope`

整个插槽的处理阶段大致分为三步：

- 编译
- 生成渲染模板
- 生成VNode

以下面代码为例，简要概述插槽运转的过程。

```html
<div id='app'>
  <test>
    <template slot="hello">
      123
    </template>
  </test>
</div>
<script>
  new Vue({
    el: '#app',
    components: {
      test: {
        template: '<h1>' +
          '<slot name="hello"></slot>' +
          '</h1>'
      }
    }
  })
</script>
```

### 父组件编译阶段

编译是将模板文件解析成`AST`语法树，会将插槽`template`解析成如下数据结构：

```javascript
{
  tag: 'test',
  scopedSlots: { // 作用域插槽
    // slotName: ASTNode,
    // ...
  }
  children: [
    {
      tag: 'template',
      // ...
      parent: parentASTNode,
      children: [ childASTNode ], // 插槽内容子节点，即文本节点123
      slotScope: undefined, // 作用域插槽绑定值
      slotTarget: "\"hello\"", // 具名插槽名称
      slotTargetDynamic: false // 是否是动态绑定插槽
      // ...
    }
  ]
}

```

### 父组件生成渲染方法

根据`AST`语法树，解析生成渲染方法字符串，最终父组件生成的结果如下所示，这个结构和我们直接写`render`方法一致，本质都是生成`VNode`, 只不过`_c`或`h`是`this.$createElement`的缩写。

```JavaScript
with(this){
  return _c('div',{attrs:{"id":"app"}},
  [_c('test',
    [
      _c('template',{slot:"hello"},[_v("\n      123\n    ")])],2)
    ],
  1)
}
```

### 父组件生成VNode

调用`render`方法，生成`VNode`,`VNode`具体格式如下：

```JavaScript
{
  tag: 'div',
  parent: undefined,
  data: { // 存储VNode配置项
    attrs: { id: '#app' }
  },
  context: componentContext, // 组件作用域
  elm: undefined, // 真实DOM元素
  children: [
    {
      tag: 'vue-component-1-test',
      children: undefined, // 组件为页面最小组成单元，插槽内容放放到子组件中解析
      parent: undefined,
      componentOptions: { // 组件配置项
        Ctor: VueComponentCtor, // 组件构造方法
        data: {
          hook: {
            init: fn, // 实例化组件调用方法
            insert: fn,
            prepatch: fn,
            destroy: fn
          },
          scopedSlots: { // 作用域插槽配置项，用于生成作用域插槽VNode
            slotName: slotFn
          }
        },
        children: [ // 组件插槽节点
          tag: 'template',
          propsData: undefined, // props参数
          listeners: undefined,
          data: {
            slot: 'hello'
          },
          children: [ VNode ],
          parent: undefined,
          context: componentContext // 父组件作用域
          // ...
        ] 
      }
    }
  ],
  // ...
}
```

在`vue`中，**组件是页面结构的基本单元**，从上述的`VNode`中，我们也可以看出，`VNode`页面层级结构结束于`test`组件，`test`组件`children`处理会在子组件初始化过程中处理。子组件构造方法组装与属性合并在**vue-dev\src\core\vdom\create-component.js** `createComponent`方法中，组件实例化调用入口是在**vue-dev\src\core\vdom\patch.js** `createComponent`方法中。

### 子组件状态初始化

实例化子组件时，会在`initRender` -> `resolveSlots`方法中，将子组件插槽节点挂载到组件作用域`vm`中，挂载形式为`vm.$slots = {slotName: [VNode]}`形式。

### 子组件编译阶段

子组件在编译阶段，会将`slot`节点，编译成以下`AST`结构：

```JavaScript
{
  tag: 'h1',
  parent: undefined,
  children: [
    {
      tag: 'slot',
      slotName: "\"hello\"",
      // ...
    }
  ],
  // ...
}
```

### 子组件生成渲染方法

生成的渲染方法如下，其中`_t`为`renderSlot`方法的简写，从`renderSlot`方法，我们就可以直观的将插槽内容与插槽点联系在一起。

```JavaScript
// 渲染方法
with(this){
  return _c('h1',[ _t("hello") ], 2)
}
```

```JavaScript
// 源码路径：vue-dev\src\core\instance\render-helpers\render-slot.js
export function renderSlot (
  name: string,
  fallback: ?Array<VNode>,
  props: ?Object,
  bindObject: ?Object
): ?Array<VNode> {
  const scopedSlotFn = this.$scopedSlots[name]
  let nodes
  if (scopedSlotFn) { // scoped slot
    props = props || {}
    if (bindObject) {
      if (process.env.NODE_ENV !== 'production' && !isObject(bindObject)) {
        warn(
          'slot v-bind without argument expects an Object',
          this
        )
      }
      props = extend(extend({}, bindObject), props)
    }
    // 作用域插槽，获取插槽VNode
    nodes = scopedSlotFn(props) || fallback
  } else {
    // 获取插槽普通插槽VNode
    nodes = this.$slots[name] || fallback
  }

  const target = props && props.slot
  if (target) {
    return this.$createElement('template', { slot: target }, nodes)
  } else {
    return nodes
  }
}
```

### 作用域插槽与具名插槽区别

```html
<!-- demo -->
<div id='app'>
  <test>
      <template slot="hello" slot-scope='scope'>
        {{scope.hello}}
      </template>
  </test>
</div>
<script>
    var vm = new Vue({
        el: '#app',
        components: {
            test: {
                data () {
                    return {
                        hello: '123'
                    }
                },
                template: '<h1>' +
                    '<slot name="hello" :hello="hello"></slot>' +
                  '</h1>'
            }
        }
    })

</script>
```

作用域插槽与普通插槽相比，主要区别在于**插槽内容可以获取到子组件作用域变量**。由于需要注入子组件变量，相比于具名插槽，作用域插槽有以下几点不同：

- 作用域插槽在组装渲染方法时，生成的是一个包含注入作用域的方法，相对于`createElement`生成`VNode`，多了一层注入作用域方法包裹，这也就决定插槽`VNode`作用域插槽是在子组件生成`VNode`时生成，而具名插槽是在父组件创建`VNode`时生成。`_u`为`resolveScopedSlots`，其作用为将节点配置项转换为`{scopedSlots: {slotName: fn}}`形式。

```JavaScript
 with (this) {
        return _c('div', {
            attrs: {
                "id": "app"
            }
        }, [_c('test', {
            scopedSlots: _u([{
                key: "hello",
                fn: function(scope) {
                    return [_v("\n        " + _s(scope.hello) + "\n      ")]
                }
            }])
        })], 1)
    }
```

- 子组件初始化时会处理具名插槽节点，挂载到组件`$slots`中，作用域插槽则在`renderSlot`中直接被调用

除此之外，其他流程大致相同。插槽作用机制不难理解，关键还是**模板解析**与**生成render函数**这两步内容较多，流程较长，比较难理解。

## 使用技巧

通过以上解析，能大概了解插槽处理流程。工作中大部分都是用模板来编写`vue`代码，但是某些时候模板有一定的局限性，需要借助于`render`方法放大`vue`的组件抽象能力。那么在`render`方法中，我们插槽的使用方法如下：

### 具名插槽

插槽处理一般分为两块：

- 父组件

父组件只需要写成模板编译成的渲染方法即可，即指定插槽`slot`名称

- 子组件

由于子组件时直接拿父组件初始化阶段生成的`VNode`，所以子组件只需要将`slot`标签替换为父组件生成的`VNode`，子组件在初始化状态时会将具名插槽挂载到组件`$slots`属性上。

```html
<div id='app'>
<!--  <test>-->
<!--    <template slot="hello">-->
<!--      123-->
<!--    </template>-->
<!--  </test>-->
</div>
<script>
  new Vue({
    // el: '#app',
    render (createElement) {
      return createElement('test', [
        createElement('h3', {
          slot: 'hello',
          domProps: {
            innerText: '123'
          }
        })
      ])
    },
    components: {
      test: {
        render(createElement) {
          return createElement('h1', [ this.$slots.hello ]);
        }
        // template: '<h1>' +
        //   '<slot name="hello"></slot>' +
        //   '</h1>'
      }
    }
  }).$mount('#app')
</script>
```

### 作用域插槽

作用域插槽使用比较灵活，可以注入子组件状态。作用域插槽 + `render`方法，对于二次组件封装作用非常大。举个栗子，在对`ElementUI` `table`组件进行基于`JSON`数据封装时，作用域插槽用处就非常大了。

```html
<div id='app'>
<!--  <test>-->
<!--    <span slot="hello" slot-scope='scope'>-->
<!--      {{scope.hello}}-->
<!--    </span>-->
<!--  </test>-->
</div>
<script>
  new Vue({
    // el: '#app',
    render (createElement) {
      return createElement('test', {
        scopedSlots:{
          hello: scope => { // 父组件生成渲染方法中，最终转换的作用域插槽方法和这种写法一致
            return createElement('span', {
              domProps: {
                innerText: scope.hello
              }
            })
          }
        }
      })
    },
    components: {
      test: {
        data () {
          return {
            hello: '123'
          }
        },
        render (createElement) {
          // 作用域插槽父组件传递过来的是function，需要手动调用生成VNode
          let slotVnode = this.$scopedSlots.hello({ hello: this.hello })
          return createElement('h1', [ slotVnode ])
        }
        // template: '<h1>' +
        //   '<slot name="hello" :hello="hello"></slot>' +
        //   '</h1>'
      }
    }
  }).$mount('#app')

</script>
```

## 小结

知其然知其所以然，方可以不变应万变。
