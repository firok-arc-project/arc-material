---
title: 一点对 Windows 自带中文输入法的优化
id: 9473cddb-d3f4-4493-8579-281727c901d4
createTimestamp: 2023-12-27T10:59:00+08:00
updateTimestamp: 2023-12-27T10:59:00+08:00
sortTimestamp: 2023-12-27T10:59:00+08:00
tags:
  - doc
  - tech
---

如果你用的是 Windows 自带输入法, 建议你没事就执行一下下面这条指令:

```bash
del /S "%AppData%\Microsoft\InputMethod\Chs\UDP*.tmp"
```

有兴趣的话你可以去看看这个目录下有什么, 不出意外的话, 可能已经堆积了几千万个不知道做何用的 `.tmp` 文件.

这些文件会不断的重新创建, 越堆越多, 在部分设备上可能导致输入法的响应变得非常非常非常缓慢.
