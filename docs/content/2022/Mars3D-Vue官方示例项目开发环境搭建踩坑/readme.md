---
title: Mars3D-Vue 官方示例项目开发环境搭建踩坑
id: 3b800025-8a7a-459c-9fbf-94c39dae4df3
createTimestamp: 2022-03-17T09:25:00+08:00
updateTimestamp: 2022-03-17T09:25:00+08:00
sortTimestamp: 2022-03-17T09:25:00+08:00
tags:
  - doc
  - tech
---

## 依赖缺失

`git clone` 自 GitHub 并 `npm install` 后, 运行 `npm run serve` 报错:

```
Error: Cannot find module 'fs-extra'
Error: Cannot find module 'serve-static'
```

运行如下指令解决:

```bash
npm install --save fs-extra serve-static
```

## ESLint 报错

手动安装依赖后可以启动开发服务器, 但是打开页面提示报错:

```
Failed to load config "standard" to extend from.
```

在 `.eslintrc.js` 中找到

```
extends: ["plugin:vue/vue3-essential", "standard", "@vue/typescript/recommended"],
```

改为

```
extends: ["plugin:vue/vue3-essential", "@vue/typescript/recommended"],
```

即可正常启动开发环境.

> 烦死 ESLint 这破玩意了

另外, 想关闭 ESLint 检查的话, 直接在 `.eslintignore` 里加一行 `*` 来禁用所有文件的语法校验吧, 太恶心了
