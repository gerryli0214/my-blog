# Vue指令实现原理

## 前言

`vue`中，

## 插槽调用流程

### 模板编译

```JavaScript
{
  tag: 'input',
  parent: ASTElement,
  directives: [
    {
      arg: null, // 参数
      end: 56, // 指令结束字符位置
      isDynamicArg: false, // 动态参数
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
            "value": (inputValue)
        },
        on: {
            "input": function($event) {
                if ($event.target.composing)
                    return;
                inputValue = $event.target.value
            }
        }
    })])
}

```
