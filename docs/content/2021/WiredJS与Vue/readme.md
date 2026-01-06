---
title: WiredJS 与 Vue
id: b782276d-e072-4fc7-a764-5b98caca5b30
createTimestamp: 2021-05-09T21:24:00+08:00
updateTimestamp: 2021-05-09T21:24:00+08:00
sortTimestamp: 2021-05-09T21:24:00+08:00
tags:
  - doc
  - tech
---

之前偶然搜到一个手绘涂鸦风的前端框架 **WiredJS**, 看起来还不错就用上了这个框架虽然在介绍里写着支持Vue, 但是实际使用体验不是那么舒服, Vue的一些双向绑定事件挂不上

> 本文写于 2021-05-09, 实际情况可能已经发生改变

WiredJS 官网 [https://wiredjs.com/](https://wiredjs.com/)
WiredJS 官方组件展示 [https://wiredjs.com/showcase.html](https://wiredjs.com/showcase.html)
WiredJS GitHub页面 [https://github.com/rough-stuff/wired-elements](https://github.com/rough-stuff/wired-elements)

## 控件参数

框架自带的控件都能在源码中找到, 大部分都继承自 `WiredBase`
看起来框架本身是 TypeScript 写的, 现在懒得学 TS 就没仔细看
简略一扫, 大部分控件可供调整的参数都写在控件类的开头

```typescript
@customElement('wired-button')
export class WiredButton extends WiredBase {
  @property({ type: Number }) elevation = 1;
  @property({ type: Boolean, reflect: true }) disabled = false;
  
  @query('button') private button?: HTMLButtonElement;
  private ro?: ResizeObserver;
  private roAttached = false;
```

从 `<wired-button>` 的部分源码可以看到这个控件有俩能调的参数 `elevation` 和 `disabled`
其中, `elevation` 在好多控件里都有, 用于调整控件渲染时的"纸片层数", 类似于控件厚度, 数字越大控件越厚

```html
<wired-button :elevation="1" :disabled="false">按钮文本</wired-button>
```

## 几个表单控件的双向绑定

WiredJS 提供的表单输入控件没法直接用 `v-model` 双向绑定, 必须手动处理数据同步

### 文本输入框

```html
<wired-input type="text" :value="someValue" @input="someValue = $event.target.value"/>
```

经测试, 输入框里的值改变的时候是能更新到内存里, 但是内存值的更新没法更新到UI上
查看官方GitHub Issue也没找到解决方案

可以用 `<wired-card>` 配合 `<input>` 来自己做一个输入框

HTML代码

```html
<wired-card id="input-box">
    <input type="text" v-model="someValue" id="input">
</wired-card>
```

CSS代码

```css
#input-box
{
    margin-bottom: 15px;
    padding: 5px;
    width: 180px;
    height: 35px
}
#input
{
    display: block;
    margin: 1px;
    width: 170px;
    height: 28px;
    border: none;
    outline: none;
    font-size: 16px;
}
```

### 搜索框

```html
<wired-search-input :value="someValue" @input="someValue = $event.target.value">
```

### 多选框

```html
<wired-checkbox :checked="someValue" @input="someValue = $event.target.checked">
```

## 自定义控件

看 WiredJS 官方文档的时候发现这玩意渲染是用的 **RoughJS**
发现用起来也很简单, 可以非常方便地自定义控件

RoughJS 官网 [https://roughjs.com/](https://roughjs.com/)
RoughJS GitHub [https://github.com/rough-stuff/rough](https://github.com/rough-stuff/rough)

下面就是一个自己写的动态绘图控件

```html
<template>
    <rough-canvas :width="width" :height="height" :options="options">
        <span v-for="compo in img">
            <rough-line v-if="compo.type==='line'"
                        :x1="compo.x1" :y1="compo.y1"
                        :x2="compo.x2" :y2="compo.y2"
            />
            <rough-rectangle v-else-if="compo.type==='rectangle'"
                             :x1="compo.x1" :y1="compo.y1"
                             :x2="compo.x2" :y2="compo.y2"
            />
            <rough-ellipse v-else-if="compo.type==='ellipse'"
                           :x="compo.x" :y="compo.y"
                           :width="compo.width" :height="compo.height"
            />
            <rough-circle v-else-if="compo.type==='circle'"
                          :x="compo.x" :y="compo.y"
                          :diameter="compo.diameter"
            />
            <rough-linear-path v-else-if="compo.type==='linear-path'"
                               :points="compo.points"
            />
            <rough-polygon v-else-if="compo.type==='polygon'"
                           :vertices="compo.vertices"
            />
            <rough-arc v-else-if="compo.type==='arc'"
                       :x="compo.x" :y="compo.y"
                       :width="compo.width" :height="compo.height"
                       :start="compo.start" :stop="compo.stop"
                       :closed="compo.closed"
            />
            <rough-curve v-else-if="compo.type==='curve'"
                         :points="compo.points"
            />
            <rough-path v-else-if="compo.type==='path'"
                        :d="compo.d"
            />
        </span>
    </rough-canvas>
</template>

<script>
    export default {
        name: "RoughImage",
        props: {
            img: { type: Array, default: [] },
            width: { type: String, default: '100px' },
            height: { type: String, default: '100px' },
            options: { type: JSON, default: ()=>{}, },
        },
    }
</script>
```
