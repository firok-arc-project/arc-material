---
title: Vite + Maven + Spring Boot 组合编译
id: facd781c-992a-461a-990f-6e468adae298
createTimestamp: 2023-11-07T10:07:00+08:00
updateTimestamp: 2023-11-07T10:07:00+08:00
sortTimestamp: 2023-11-07T10:07:00+08:00
tags:
  - doc
  - tech
---

假设项目目录结构如下:

```plaintext
/project/
  /backend/ # 后端目录
    /pom.xml
    /src/
      /main/
        /resources/
          /static/ # 后端静态资源目录
  /frontend/ # 前端目录
    /package.json
    /vite.config.js
```

在 `vite.config.js` 的 `defineConfig` 中新增如下项:

```
build: {
  outDir: '../backend/src/main/resources/static',
  emptyOutDir: true,
},
```

> 如果将 `outDir` 设为前端根目录之外的目录, 打包前将默认不会清空输出目录. 此时将 `emptyOutDir` 设为 `true` 即可

在 `package.json` 的 script 中增加如下项:

```json
{
  ...
  "script": {
    ...
    "aio": "chcp 65001 && npm run build && cd ../backend && call mvn clean package"
  }
}
```

> 先 `chcp` 是为了减少乱码发生的可能性. 如果你的项目没用 unicode 编码 (那得是多邪恶的项目🤣🤜😈) 那就删掉这个

做完之后, 需要编译打包整个项目的时候直接在 `frontend` 目录执行 `npm run aio` 就行了, 打包之后的内容会输出到 `backend/target` 下.

更多的, 如果你想调整 Maven 的打包输出目录那就在 `pom.xml` 里加上如下内容:

```xml
...
<build>
  ...
  <directory>../anything-u-want</directory>
</build>
```
