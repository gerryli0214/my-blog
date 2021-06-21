　　在使用vue的过程中，经常会用到`Vue.use`，但是大部分对它一知半解，不了解在调用的时候具体做了什么，因此，本文简要概述下在vue中，如何封装自定义插件。

　　在开始之前，先补充一句，其实利用`Vue`封装自定义插件的本质就是组件实例化的过程或者指令等公共属性方法的定义过程，比较大的区别在于封装插件需要手动干预，就是一些实例化方法需要手动调用，而`Vue`的实例化，很多逻辑内部已经帮忙处理掉了。插件相对于组件的优势就是插件封装好了之后，可以开箱即用，而组件是依赖于项目的。对组件初始化过程不是很熟悉的可以参考这篇博文。

　　我们从vue源码中，可以看到`Vue.use`的方法定义如下：　
```JavaScript
Vue.use = function (plugin: Function | Object) {
    const installedPlugins = (this._installedPlugins || (this._installedPlugins = []))
    // 已经存在插件，则直接返回插件对象
    if (installedPlugins.indexOf(plugin) > -1) {
      return this
    }

    // additional parameters
    const args = toArray(arguments, 1)
    args.unshift(this)
    // vue插件形式可以是对象，也可以是方法，默认会传递一个Vue的构造方法作为参数
    if (typeof plugin.install === 'function') {
      plugin.install.apply(plugin, args)
    } else if (typeof plugin === 'function') {
      plugin.apply(null, args)
    }
    installedPlugins.push(plugin)
    return this
  }
```
　　从上述代码中，我们可以看出，`Vue.use`代码比较简洁，处理逻辑不是很多。我们常用的`Vue.use(xxx)`，xxx可以是方法，也可以是对象。在`Vue.use`中，通过`apply`调用插件方法，传入一个参数，Vue的构造方法。举个栗子，最简单的Vue插件封装如下：

```JavaScript
// 方法
function vuePlugins (Vue) {
    Vue.directive('test', {
        bind (el) {
            el.addEventListener('click', function (e) {
                alert('hello world')
            })
        }
    })
}
```

```JavaScript
// 对象
const vuePlugins = {
    install (Vue) {
        Vue.directive('test', {
            bind (el) {
                el.addEventListener('click', function (e) {
                    alert('hello world')
                })
            }
        })
    }
}
```
　　以上两种封装方法都可以，说白了，就是将全局注册的指令封装到一个方法中，在`Vue.use`时调用。这个比较显然易懂。现在举一个稍微复杂点的例子，`tooltip`在前端开发中经常会用到，直接通过方法能够调用显示，防止不必要的组件注册引入，如果我们单独封装一个`tooltip`组件，应该如何封装呢？这种封装方式需要了解组件的初始化过程。区别在于将组件封装成插件时，不能通过`template`将组件实例化挂载到真实`DOM`中，这一步需要手动去调用对应组件实例化生命周期中的方法。具体实现代码如下：　　
```JavaScript
// component
let toast = {
    props: {
        show: {
            type: Boolean,
            default: false
        },
        msg: {
            type: String
        }
    },
    template: '<div v-show="show" class="toast">{{msg}}</div>'
}
```
## 组件初始化过程：
```JavaScript
// JavaScript初始化逻辑
// 获取toast构造实例
const TempConstructor = Vue.extend(toast)
// 实例化toast
let instance = new TempConstructor()
// 手动创建toast的挂载容器
let div = document.createElement('div')
// 解析挂载toast
instance.$mount(div)
// 将toast挂载到body中
document.body.append(instance.$el)
// 将toast的调用封装成一个方法，挂载到Vue的原型上
Vue.prototype.$toast = function (msg) {
    instance.show = true
    instance.msg = msg
    setTimeout(() => {
        instance.show = false
    }, 5000)
}
```
　　组件的定义，和普通的组件声明一致。组件的插件化过程，和普通的组件实例化一致，区别在于插件化时组件部分初始化方法需要手动调用。比如：

1. `Vue.extend`作用是组装组件的构造方法`VueComponent`

2. `new TempConstructor()`是实例化组件实例。实例化构造方法，只是对组件的状态数据进行了初始化，并没有解析组件的`template`，也没有后续的生成`vnode`和解析`vnode`

3. `instance.$mount(div)`的作用是解析模板文件，生成`render`函数，进而调用`createElement`生成`vnode`，最后生成真实DOM,将生成的真实DOM挂载到实例`instance`的`$el`属性上，也就是说，实例`instance.$el`即为组件实例化最终的结果。

4. 组件中，`props`属性最终会声明在组件实例上，所以直接通过实例的属性，也可以响应式的更改属性的传参。组件的属性初始化方法如下：

```JavaScript
function initProps (Comp) {
  const props = Comp.options.props
  for (const key in props) {
    proxy(Comp.prototype, `_props`, key)
  }
}
```

```JavaScript
// 属性代理，从一个原对象中拿数据
export function proxy (target: Object, sourceKey: string, key: string) {
  // 设置对象属性的get/set,将data中的数据代理到组件对象vm上
  sharedPropertyDefinition.get = function proxyGetter () {
    return this[sourceKey][key]
  }
  sharedPropertyDefinition.set = function proxySetter (val) {
    this[sourceKey][key] = val
  }
  Object.defineProperty(target, key, sharedPropertyDefinition)
}
```

　　从上述可以看出，最终会在构造方法中，给所有的属性声明一个变量，本质上是读取`_props`中的内容，`_props`中的属性，会在实例化组件,`initState`中的`InitProps`中进行响应式的声明，具体代码如下：

```JavaScript
function initProps (vm: Component, propsOptions: Object) {
  const propsData = vm.$options.propsData || {}
  const props = vm._props = {}
  // cache prop keys so that future props updates can iterate using Array
  // instead of dynamic object key enumeration.
  const keys = vm.$options._propKeys = []
  const isRoot = !vm.$parent
  // root instance props should be converted
  if (!isRoot) {
    toggleObserving(false)
  }
  for (const key in propsOptions) {
    keys.push(key)
    const value = validateProp(key, propsOptions, propsData, vm)
    /* istanbul ignore else */
    if (process.env.NODE_ENV !== 'production') {
      const hyphenatedKey = hyphenate(key)
      if (isReservedAttribute(hyphenatedKey) ||
          config.isReservedAttr(hyphenatedKey)) {
        warn(
          `"${hyphenatedKey}" is a reserved attribute and cannot be used as component prop.`,
          vm
        )
      }
      defineReactive(props, key, value, () => {
        if (!isRoot && !isUpdatingChildComponent) {
          warn(
            `Avoid mutating a prop directly since the value will be ` +
            `overwritten whenever the parent component re-renders. ` +
            `Instead, use a data or computed property based on the prop's ` +
            `value. Prop being mutated: "${key}"`,
            vm
          )
        }
      })
    } else {
      defineReactive(props, key, value)
    }
    // static props are already proxied on the component's prototype
    // during Vue.extend(). We only need to proxy props defined at
    // instantiation here.
    if (!(key in vm)) {
      proxy(vm, `_props`, key)
    }
  }
  toggleObserving(true)
}
```
　　这里会遍历所有订单`props`，响应式的声明属性的`get`和`set`。当对属性进行读写时，会调用对应的`get/set`，进而会触发视图的更新，`vue`的响应式原理在后面的篇章中会进行介绍。这样，我们可以通过方法参数的传递，来动态的去修改组件的`props`，进而能够将组件插件化。

　　有些人可能会有疑问，到最后挂载到`body`上的元素是通过`document.createElement('div')`创建的`div`，还是模板的`template`解析后的结果。其实，最终挂载只是组件解析后的结果。在调用`__patch__`的过程中，执行流程是，首先，记录老旧的节点，也就是`$mount(div)`中的`div`；然后，根据模板解析后的`render`生成的`vnode`的节点，去创建`DOM`节点，创建后的`DOM`节点会放到`instance.$el`中；最后一步，会将老旧节点给移除掉。所以，在我们封装一个插件的过程中，实际上手动创建的元素只是一个中间变量，并不会保留在最后。可能大家还会注意到，插件实例化完成后的`DOM`挂载也是我们手动挂载的，执行的代码是`document.body.append(instance.$el)`。

　　附：test.html 测试代码
```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <style>
    .toast{
      position: absolute;
      left: 45%;
      top: 10%;
      width: 10%;
      height: 5%;
      background: #ccc;
      border-radius: 5px;
    }
  </style>
  <title>Hello World</title>
  <script src="https://cdn.bootcss.com/vue/2.6.10/vue.js"></script>
</head>
<body>
<div id='app' v-test>
  <button @click="handleClick">我是按钮</button>
</div>
<script>
function vuePlugins (Vue) {
    Vue.directive('test', {
        bind (el) {
            el.addEventListener('click', function (e) {
                alert('hello world')
            })
        }
    })
}
// const vuePlugins = {
//     install (Vue) {
//         Vue.directive('test', {
//             bind (el) {
//                 el.addEventListener('click', function (e) {
//                     alert('hello world')
//                 })
//             }
//         })
//     }
// }
Vue.use(vuePlugins)
let toast = {
    props: {
        show: {
            type: Boolean,
            default: false
        },
        msg: {
            type: String
        }
    },
    template: '<div v-show="show" class="toast">{{msg}}</div>'
}
// 获取toast构造实例
const TempConstructor = Vue.extend(toast)
// 实例化toast
let instance = new TempConstructor()
// 手动创建toast的挂载容器
let div = document.createElement('div')
// 解析挂载toast
instance.$mount(div)
// 将toast挂载到body中
document.body.append(instance.$el)
// 将toast的调用封装成一个方法，挂载到Vue的原型上
Vue.prototype.$toast = function (msg) {
    instance.show = true
    instance.msg = msg
    setTimeout(() => {
        instance.show = false
    }, 5000)
}
var vm = new Vue({
    el: '#app',
    data: {
        msg: 'Hello World',
        a: 11
    },
    methods: {
        test () {
            console.log('这是一个主方法')
        },
        handleClick () {
            this.$toast('hello world')
        }
    },
    created() {
        console.log('执行了主组件上的created方法')
    },
})
</script>
</body>
</html>
```