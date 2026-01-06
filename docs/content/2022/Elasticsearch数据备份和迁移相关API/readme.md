---
title: Elasticsearch 数据备份和迁移相关 API
id: 05469980-ed1b-4b13-b87b-ff93f713cc96
createTimestamp: 2022-09-23T14:20:00+08:00
updateTimestamp: 2022-09-23T14:20:00+08:00
sortTimestamp: 2022-09-23T14:20:00+08:00
tags:
  - doc
  - tech
---

## 创建 ES 快照仓库

```text
POST http://localhost:9200/_snapshot/ss
{
    "type": "fs", 
    "settings": {
        "location": "/usr/share/elasticsearch/snapshot",
        "max_snapshot_bytes_per_sec" : "100mb", 
        "max_restore_bytes_per_sec" : "100mb"
    }
}
```

当 ES 实例运行在 Docker 内时,
`settings.location` 需为 Docker 容器内路径.
快照创建完成后需使用

```bash
docker cp {CONTAINER_ID}:{SNAPSHOT_PATH} {FILESYSTEM_PATH}
```

将创建好的快照文件从容器内复制出来.

[相关资料](https://www.elastic.co/guide/cn/elasticsearch/guide/current/backing-up-your-cluster.html)

> 如果 **需要恢复数据的 ES 实例** 和 **需要从快照备份数据的 ES 实例** 不是同一个,
> 在 **需要从快照备份数据的 ES 实例** 里也得执行一遍这个指令,
> 然后把快照数据复制到那个目录下

## 快照所有打开的索引

```text
PUT http://localhost:9200/_snapshot/ss/220923?wait_for_completion=true
```

这个操作是耗时操作,
添加 `wait_for_completion=true` 参数,
这样此次 HTTP 请求会在快照创建完成后才完成.

## 从快照恢复索引数据

```text
POST http://localhost:9200/_snapshot/ss/220923/_restore?wait_for_completion=true
```

## 删除所有索引

```text
DELETE http://localhost:9200/_all
```

## 其它问题

创建快照不需要停止各类服务;
从快照恢复数据时需要先把连接到这个 ES 实例的后台服务关掉,
然后再执行恢复索引数据指令.

> 恢复之前把数据全删了也行
>
