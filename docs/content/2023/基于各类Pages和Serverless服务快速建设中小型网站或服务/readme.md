---
title: 基于各类 Pages 和 Serverless 服务快速建设中小型网站或服务
id: 3d71c18a-b719-4d24-975e-260318357b5d
createTimestamp: 2023-06-20T00:23:00+08:00
updateTimestamp: 2023-06-20T00:23:00+08:00
sortTimestamp: 2023-06-20T00:23:00+08:00
tags:
  - doc
  - tech
---

基于各类 Pages 服务托管网站静态资源,
配合各类 Serverless 服务即可快速构建中小型网站.

前端的搭建与一般 Web 项目无异, 选定一个框架 (如 Vite, Nuxt.js 等) 正常开发即可;
后端开发基于标准的 [Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API) 开发 (不同运营商对传入的 `Request` 和 `Response` 有一定补强, 比如 Cloudflare 和 Vercel 都会在 `Request` 的 `Headers` 里附加访问者真实 IP 等信息).

下面列出的是截至 2023-06-19 找到 + 体验过 **免费** 提供上述服务的运营商

## Pages 和 Serverless 服务

| 运营商     | Pages 服务 | Serverless 服务 | 备注                                                       |
| ---------- | ---------- | --------------- | ---------------------------------------------------------- |
| GitHub     | 有         | 无              | 非 GitHub Pro 订阅用户只能基于 public repo 搭建 Pages 服务 |
| Cloudflare | 有         | 有              | (未测试不同类型 repo 的影响)                               |
| Vercel     | 有         | 有              | 可以基于 private repo 构建 Pages 和 Serverless 服务        |

GitHub 官方只提供了快速基于 repo 中的 Markdown 文档或项目编译打包成果建立 Pages 的 [文档](https://docs.github.com/zh/pages). 更多支持需要自行调配 [GitHub Actions](https://docs.github.com/zh/actions) 完成.

Cloudflare Pages 提供了从各类常见 Web 框架快速建设 Pages 服务的 [文档](https://developers.cloudflare.com/pages), 支持 Pages 和 Serverless 服务共存于同项目.

Vercel 功能与 Cloudflare 类似, 其中 Serverless 功能分为 [Serverless Functions](https://vercel.com/docs/concepts/functions/serverless-functions) 和 [Edge Functions](https://vercel.com/docs/concepts/functions/edge-functions), 官方描述了 [两者的区别](https://vercel.com/docs/concepts/limits/overview#functions-comparison).

另外还找到三家运营商没实际体验过, 有兴趣的可以试试:

* [Render.com](https://render.com/)
* [Netlify](https://www.netlify.com/)
* ~~[Mogenius](https://mogenius.com/)~~ 据说有免费额度, 但是看商店页面好像只有 14 天试用

## 数据持久化

Cloudflare 和 Vercel 都提供了免费的数据库功能.

| 运营商     | 关系型数据库                                                                   | 非关系型数据库                           | 键值存储                                                      | 对象存储                                                                                | 图片优化                                                               |
| ---------- | ------------------------------------------------------------------------------ | ---------------------------------------- | ------------------------------------------------------------- | --------------------------------------------------------------------------------------- | ---------------------------------------------------------------------- |
| Cloudflare | [D1](https://developers.cloudflare.com/d1/) _(当前处于 Alpha 阶段, 文档并不全)_ | [R2](https://developers.cloudflare.com/r2/) | [KV](https://developers.cloudflare.com/workers/runtime-apis/kv/) | [Durable Objects](https://developers.cloudflare.com/workers/runtime-apis/durable-objects/) | [Image Optimization](https://developers.cloudflare.com/images/)           |
| Vercel     | [Postgres](https://vercel.com/docs/storage/vercel-postgres)                       | 无                                       | [KV](https://vercel.com/docs/storage/vercel-kv)                  | 无                                                                                      | [Image Optimization](https://vercel.com/docs/concepts/image-optimization) |

个人用过的服务, 体验感知如下

* Cloudflare KV
  * 免费用户写入和读取存储次数有限 (1000 次 / 10 万次)
  * 免费用户数据库最大容量 1GB
* ~~Cloudflare D1~~
  * ~~文档不全, 没有进行详细测试~~
* Vercel Postgres
  * CPU 时间以 **数据库活跃状态** 计算
    * 每次请求会使数据库进入 active 状态
    * 5 分钟没有请求, 数据库进入 inactive 状态
    * 免费用户有 240 小时 CPU 时间
  * 免费用户最大只能创建 1 个数据库
    * 虽然把多个服务的表放在一个数据库里不是不行
  * 免费用户数据库最大容量 256 MB

这两家运营商提供的免费额度都不小,
完全可以用于搭建个人博客之类的小型服务.
但是使用量稍微高一些就可能爆表,
且由于免费额度是账户共享,
额度爆表就可能同时影响所有项目的使用.

## 访问优化

在不进行任何优化的情况下, 上述三家运营商的主域名均无法流畅访问.

目前可选方案之一是购买自定义域名, 然后用 Cloudflare DNS 解析 + 代理流量.

如果购买域名为 `example.com`, 部署在 Vercel 下的服务域名为 `xxx.username.vercel.app`, 可在 Cloudflare DNS 添加一条 `CNAME` 记录并启动代理, 配置值为 `xxx` -> `xxx.username.vercel.app`, 然后即可通过 `xxx.example.com` 域名访问服务.
