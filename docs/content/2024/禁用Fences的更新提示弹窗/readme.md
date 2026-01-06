---
title: 禁用 Fences 的更新提示弹窗
id: a269bbf8-a8ed-4af4-a16b-7df82424b876
createTimestamp: 2024-10-11T01:02:00+08:00
updateTimestamp: 2024-10-11T01:02:00+08:00
sortTimestamp: 2024-10-11T01:02:00+08:00
tags:
  - doc
  - tech
---

使用如下命令行以禁用 Fences 的更新弹窗:

```bash
reg add HKCU\Software\Stardock\Settings /v SupressAutoUpdate /t REG_DWORD /d "0"
```


> 资料来源
> https://forums.stardock.com/486084/fences-support-faq#disablefences

