---
title: Vue3 渲染函数和虚拟节点应用中的一些细节
id: 436cebe3-75ff-43c2-8bf0-c186ae48171f
createTimestamp: 2026-01-23T01:00:00+08:00
updateTimestamp: 2026-01-23T01:00:00+08:00
sortTimestamp: 2026-01-23T01:00:00+08:00
tags:
  - doc
  - tech
---

这篇文章是关于 Vue3 的渲染函数 (函数式组件) 应用的一些细节,
如果你还不了解渲染函数,
请先阅读并理解 [官方文档](https://cn.vuejs.org/guide/extras/render-function) 的内容.

## 模版式函数与函数式组件的交互

在模版式组件中调用函数式组件时,
Vue3 将会为编写的函数式组件传递两个参数:

1. `props`: 在模版式组件中使用函数式组件时, 在 dom 上挂载的所有属性 (attribution)
2. `context`: 一个上下文对象, 主要包含:
   1. 可能从模版函数提供的插槽内容
   2. emit 函数
   3. expose 数据

不论是在模版式组件还是函数式组件中,
所有的插槽在底层都是一个 [**返回 vnodes 数组的函数**](https://cn.vuejs.org/guide/extras/render-function#rendering-slots).

这个东西的定义基本上是下面这样:

```typescript
import type {SetupContext, VNodeArrayChildren} from '@vue/runtime-core'

type SlotFunction = (
  slotFunctionProps: any,
) => VNodeArrayChildren
```

如果需要在模版式组件中为函数式组件传递插槽内容, 用法大概率是这样:

```html
<template>
  <div>
    <functional-component>
      
      <template #abc="slotFunctionProps: any">
        <div>
          {{ slotFunctionProps }}
        </div>
      </template>
      
    </functional-component>
  </div>
</template>

<script setup lang="ts">
import { FunctionalComponent } from './functional-component.ts'

</script>
```

底层的函数式组件可能会这样编写:

```typescript

import {
  type ComponentObjectPropsOptions, type RendererNode,
  type SetupContext,
  type VNode,
  type VNodeArrayChildren,
} from '@vue/runtime-core'

export function FunctionalComponent(
  props: any,
  context: SetupContext,
): VNode<RendererNode, RendererElement, MarkedAreaProps>
{
  const slots = context.slots
  
  const slotFunction: SlotFunction = slots['abc']
  
  return h('div', {}, [
    ...slotFunction({
      'key123': 'value123',
    })
  ])
}

```

## 在模版式组件中直接渲染 VNode 数据

在部分情况下,
我们可能需要将通过某些方式渲染出的 VNode 对象 / 数组直接渲染在模版式组件中,
这种情况需要使用内置的 `<component>` 组件完成.

对于 **单个 VNode 对象或 VNode 数组**:

```html
<component :is="() => nodes" />
```

(为 `is` 属性绑定一个 **返回 VNode 对象或数组的函数**; 在这里是直接写成一个 lambda 函数)

对于 **单个 VNode 对象**, 可以这样写:

```html
<component :is="node" />
```

(直接为 `is` 属性绑定一个 VNode 对象)

## 在函数式组件中直接渲染 HTML 内容

在部分情况下,
我们可能需要在函数式组件中直接渲染一些原始 HTML,
这种情况直接在 `h()` 函数传参将需要渲染的 HTML 数据作为 `innerHTML` 参数即可:

```typescript
function FunctionalComponent(props: any): VNodeArrayChildren
{
  const html: string = props.html ?? '<span> 123 </span>'
  return [
    h('div', { innerHTML: html })
  ]
}
```
