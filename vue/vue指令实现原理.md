# Vue指令实现原理

## 前言

自定义指令是`vue`中使用频率仅次于组件，其包含`bind`、`inserted`、`update`、`componentUpdated`、`unbind`五个生命周期钩子。本文将对`vue`指令的工作原理进行相应介绍，从本文中，你将得到：

- 指令的工作原理
- 指令使用的注意事项

## 基本使用

官网案例：

```html
<div id='app'>
  <input type="text" v-model="inputValue" v-focus>
</div>
<script>
  Vue.directive('focus', {
    // 第一次绑定元素时调用
    bind () {
      console.log('bind')
    },
    // 当被绑定的元素插入到 DOM 中时……
    inserted: function (el) {
      console.log('inserted')
      el.focus()
    },
    // 所在组件VNode发生更新时调用
    update () {
      console.log('update')
    },
    // 指令所在组件的 VNode 及其子 VNode 全部更新后调用
    componentUpdated () {
      console.log('componentUpdated')
    },
    // 只调用一次，指令与元素解绑时调用
    unbind () {
      console.log('unbind')
    }
  })
  new Vue({
    data: {
      inputValue: ''
    }
  }).$mount('#app')
</script>
```

## 指令工作原理

### 初始化

初始化全局`API`时，在`platforms/web`下，调用`createPatchFunction`生成`VNode`转换为真实`DOM`的`patch`方法，初始化中比较重要一步是定义了与`DOM`节点相对应的`hooks`方法，在`DOM`的创建(`create`)、激活(`avtivate`)、更新(`update`)、移除(`remove`)、销毁(`destroy`)过程中，分别会轮询调用对应的`hooks`方法，这些`hooks`中一部分是指令声明周期的入口。

```JavaScript
// src/core/vdom/patch.js
const hooks = ['create', 'activate', 'update', 'remove', 'destroy']
export function createPatchFunction (backend) {
  let i, j
  const cbs = {}

  const { modules, nodeOps } = backend
  for (i = 0; i < hooks.length; ++i) {
    cbs[hooks[i]] = []
    // modules对应vue中模块，具体有class, style, domListener, domProps, attrs, directive, ref, transition
    for (j = 0; j < modules.length; ++j) {
      if (isDef(modules[j][hooks[i]])) {
        // 最终将hooks转换为{hookEvent: [cb1, cb2 ...], ...}形式
        cbs[hooks[i]].push(modules[j][hooks[i]])
      }
    }
  }
  // ....
  return function patch (oldVnode, vnode, hydrating, removeOnly) {
    // ...
  }
}
```

```
```

### 模板编译

模板编译就是解析指令参数，具体解构后的`ASTElement`如下所示：

```JavaScript
{
  tag: 'input',
  parent: ASTElement,
  directives: [
    {
      arg: null, // 参数
      end: 56, // 指令结束字符位置
      isDynamicArg: false, // 动态参数,v-xxx[dynamicParams]='xxx'形式调用
      modifiers: undefined, // 指令修饰符
      name: "model",
      rawName: "v-model", // 指令名称
      start: 36, // 指令开始字符位置
      value: "inputValue" // 模板
    },
    {
      arg: null,
      end: 67,
      isDynamicArg: false,
      modifiers: undefined,
      name: "focus",
      rawName: "v-focus",
      start: 57,
      value: ""
    }
  ],
  // ...
}

```

### 生成渲染方法

`vue`推荐采用指令的方式去操作`DOM`，由于自定义指令可能会修改`DOM`或者属性，所以避免指令对模板解析的影响，在生成渲染方法时，首先处理的是指令，如`v-model`，本质是一个语法糖，在拼接渲染函数时，会给元素加上`value`属性与`input`事件（以`input`为例，这个也可以用户自定义）。

```JavaScript
with (this) {
    return _c('div', {
        attrs: {
            "id": "app"
        }
    }, [_c('input', {
        directives: [{
            name: "model",
            rawName: "v-model",
            value: (inputValue),
            expression: "inputValue"
        }, {
            name: "focus",
            rawName: "v-focus"
        }],
        attrs: {
            "type": "text"
        },
        domProps: {
            "value": (inputValue) // 处理v-model指令时添加的属性
        },
        on: {
            "input": function($event) { // 处理v-model指令时添加的自定义事件
                if ($event.target.composing)
                    return;
                inputValue = $event.target.value
            }
        }
    })])
}

```

### 生成VNode

`vue`的指令设计是方便我们操作`DOM`,在生成`VNode`时，指令并没有做额外处理。

### 生成真实DOM

在`vue`初始化过程中，我们需要记住两点：

- 状态的初始化是 父 -> 子，如`beforeCreate`、`created`、`beforeMount`，调用顺序是 父 -> 子
- 真实`DOM`挂载顺序是 子 -> 父，如`mounted`，这是因为在生成真实`DOM`过程中，如果遇到组件，会走组件创建的过程，真实`DOM`的生成是从子到父一级级拼接。

在`patch`过程中，每此调用`createElm`生成真实`DOM`时，都会检测当前`VNode`是否存在`data`属性，存在，则会调用`invokeCreateHooks`，走初创建的钩子函数，核心代码如下：

```JavaScript
// src/core/vdom/patch.js
function createElm (
    vnode,
    insertedVnodeQueue,
    parentElm,
    refElm,
    nested,
    ownerArray,
    index
  ) {
    // ...
    // createComponent有返回值，是创建组件的方法，没有返回值，则继续走下面的方法
    if (createComponent(vnode, insertedVnodeQueue, parentElm, refElm)) {
      return
    }

    const data = vnode.data
    // ....
    if (isDef(data)) {
        // 真实节点创建之后，更新节点属性，包括指令
        // 指令首次会调用bind方法，然后会初始化指令后续hooks方法
        invokeCreateHooks(vnode, insertedVnodeQueue)
    }
    // 从底向上，依次插入
    insert(parentElm, vnode.elm, refElm)
    // ...
  }
```

以上是指令钩子方法的第一个入口，是时候揭露`directive.js`神秘的面纱了，核心代码如下：

```JavaScript
// src/core/vdom/modules/directives.js

// 默认抛出的都是updateDirectives方法
export default {
  create: updateDirectives,
  update: updateDirectives,
  destroy: function unbindDirectives (vnode: VNodeWithData) {
    // 销毁时，vnode === emptyNode
    updateDirectives(vnode, emptyNode)
  }
}

function updateDirectives (oldVnode: VNodeWithData, vnode: VNodeWithData) {
  if (oldVnode.data.directives || vnode.data.directives) {
    _update(oldVnode, vnode)
  }
}

function _update (oldVnode, vnode) {
  const isCreate = oldVnode === emptyNode
  const isDestroy = vnode === emptyNode
  const oldDirs = normalizeDirectives(oldVnode.data.directives, oldVnode.context)
  const newDirs = normalizeDirectives(vnode.data.directives, vnode.context)
  // 插入后的回调
  const dirsWithInsert = [
  // 更新完成后回调
  const dirsWithPostpatch = []

  let key, oldDir, dir
  for (key in newDirs) {
    oldDir = oldDirs[key]
    dir = newDirs[key]
    // 新元素指令，会执行一次inserted钩子方法
    if (!oldDir) {
      // new directive, bind
      callHook(dir, 'bind', vnode, oldVnode)
      if (dir.def && dir.def.inserted) {
        dirsWithInsert.push(dir)
      }
    } else {
      // existing directive, update
      // 已经存在元素，会执行一次componentUpdated钩子方法
      dir.oldValue = oldDir.value
      dir.oldArg = oldDir.arg
      callHook(dir, 'update', vnode, oldVnode)
      if (dir.def && dir.def.componentUpdated) {
        dirsWithPostpatch.push(dir)
      }
    }
  }

  if (dirsWithInsert.length) {
    // 真实DOM插入到页面中，会调用此回调方法
    const callInsert = () => {
      for (let i = 0; i < dirsWithInsert.length; i++) {
        callHook(dirsWithInsert[i], 'inserted', vnode, oldVnode)
      }
    }
    // VNode合并insert hooks
    if (isCreate) {
      mergeVNodeHook(vnode, 'insert', callInsert)
    } else {
      callInsert()
    }
  }

  if (dirsWithPostpatch.length) {
    mergeVNodeHook(vnode, 'postpatch', () => {
      for (let i = 0; i < dirsWithPostpatch.length; i++) {
        callHook(dirsWithPostpatch[i], 'componentUpdated', vnode, oldVnode)
      }
    })
  }

  if (!isCreate) {
    for (key in oldDirs) {
      if (!newDirs[key]) {
        // no longer present, unbind
        callHook(oldDirs[key], 'unbind', oldVnode, oldVnode, isDestroy)
      }
    }
  }
}

```

对于首次创建，执行过程如下：

1. `oldVnode === emptyNode`，`isCreate`为`true`，调用当前元素中所有`bind`钩子方法。
2. 检测指令中是否存在`inserted`钩子，如果存在，则将`insert`钩子合并到`VNode.data.hooks`属性中。
3. `DOM`挂载结束后，会执行`invokeInsertHook`，所有已挂载节点，如果`VNode.data.hooks`中存在`insert`钩子。则会调用，此时会触发指令绑定的`inserted`方法。

一般首次创建只会走`bind`和`inserted`方法，而`update`和`componentUpdated`则与`bind`和`inserted`对应。在组件依赖状态发生改变时，会用`VNode diff`算法，对节点进行打补丁式更新，其调用流程：

1. 响应式数据发生改变，调用`dep.notify`，通知数据更新。
2. 调用`patchVNode`，对新旧`VNode`进行差异化更新，并全量更新当前`VNode`属性（包括指令，就会进入`updateDirectives`方法）。
3. 如果指令存在`update`钩子方法，调用`update`钩子方法，并初始化`componentUpdated`回调，将`postpatch hooks`挂载到`VNode.data.hooks`中。
4. 当前节点及子节点更新完毕后，会触发`postpatch hooks`，即指令的`componentUpdated`方法

核心代码如下：

```JavaScript
// src/core/vdom/patch.js
function patchVnode (
    oldVnode,
    vnode,
    insertedVnodeQueue,
    ownerArray,
    index,
    removeOnly
  ) {
    // ...
    const oldCh = oldVnode.children
    const ch = vnode.children
    // 全量更新节点的属性
    if (isDef(data) && isPatchable(vnode)) {
      for (i = 0; i < cbs.update.length; ++i) cbs.update[i](oldVnode, vnode)
      if (isDef(i = data.hook) && isDef(i = i.update)) i(oldVnode, vnode)
    }
    // ...
    if (isDef(data)) {
    // 调用postpatch钩子
      if (isDef(i = data.hook) && isDef(i = i.postpatch)) i(oldVnode, vnode)
    }
  }
```

`unbind`方法是在节点销毁时，调用`invokeDestroyHook`，这里不做过多描述。

## 注意事项

使用自定义指令时，和普通模板数据绑定，`v-model`还是存在一定的差别，如虽然我传递参数（`v-xxx='param'`）是一个引用类型，数据变化时，并不能触发指令的`bind`或者`inserted`，这是因为在指令的声明周期内，`bind`和`inserted`只是在初始化时调用一次，后面只会走`update`和`componentUpdated`。

指令的声明周期执行顺序为`bind -> inserted -> update -> componentUpdated`，如果指令需要依赖于子组件的内容时，推荐在`componentUnpdated`中写相应业务逻辑。

`vue`中，很多方法都是循环调用，如`hooks`方法，事件回调等，一般调用都用`try catch`包裹，这样做的目的是为了防止一个处理方法报错，导致整个程序崩溃，这一点在我们开发过程中可以借鉴使用。

## 小结

开始看整个`vue`源码时，对很多细枝末节方法都不怎么了解，通过梳理具体每个功能的实现时，渐渐能够看到整个`vue`全貌，同时也能避免开发使用中的一些坑点。