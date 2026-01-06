---
title: 纯前端编辑文件数据
id: e9453db2-ae6e-4adb-afc8-cc30584c9192
createTimestamp: 2020-11-12T16:43:00+08:00
updateTimestamp: 2020-11-12T16:43:00+08:00
sortTimestamp: 2020-11-12T16:43:00+08:00
tags:
  - doc
  - tech
---

直接上代码

```javascript
const { log } = console;

// 纯代码创建文件并传给后台
let blob = new Blob( [JSON.stringify( {hello: "world"} )], {type: 'application/json'} );
let file = new File([blob],'test.txt');
// 发送文件
let formData = new FormData();
formData.append('file',file);
axios.post(
    'http://192.168.80.167:28086/api/attachment/test',
    formData,
    {
        headers: {
            "Authorization": "Bearer 0000",
            "Content-Type": "multipart/form-data",
        },
    }
);

// 纯代码创建文件并下载到用户本地
let data = JSON.stringify( { hello: 'world' } );
data = new Blob([data]);
// 创建一个 <a> 元素, 用于模拟用户的点击下载操作
let domLink = document.createElement('a');

let url = window.URL.createObjectURL(data);
domLink.href = url; // 文件数据
domLink.download = 'what-you-want.json'; // 文件名
domLink.click() // 模拟用户点击
```
