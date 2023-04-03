---
uuid: eee38010-b64c-11ed-aa77-57aed15672db
author: dujun
email: dujun05@kuaishou.com
github: https://github.com/dujun
avatar: https://avatars.githubusercontent.com/u/119213?v=4
title: next/image 使用是收费的??
date: 2023-02-27 11:15:05
tags:
---

> 起因: 新项目上线一段时间后,发现在 Vercel Usage 里的 Image Optimization 增长迅速,很快就超过了 5000 的免费额度,产生了额外的费用,对小团队来说还是能省则省吧。

刚开始怀疑是 cdn 的问题,调整了一番,数据还在持续增长;又发现有个用作用户 sbt 的背景图体积特别大,压缩后重新部署,观察了一天还是不断增长。

接下来就仔细研读 nextjs 的文档了,发现是因为 next/image 给做了一些优化,那它是如何收费的?又做了什么优化?要怎么调整呢?

#### 如何收费?

个人版 免费额度为 1000 个图片,超出后不再优化,但账户可能被封锁;
pro 版 免费额度为 5000 个图片,超过后每 1000 个图片收费 5 美元。

图片数量计算是根据图片的 url,一个计费周期内只计算一次,无论请求多少次。

[定价说明](https://vercel.com/docs/concepts/image-optimization/limits-and-pricing)

#### next/image 做了哪些优化

1. 缓存
   图片加载后会自动缓存在 Vercel Edge Network,类似于 cdn

2. 延迟加载

3. 图片大小/质量的优化

#### 如何调整

在我们的项目中,可以加载用户的 nft,badge 等个性数据,随着用户增多,就增加了很多图片,所以最后采取的方案就是在这些模块的 next/image 上加上 unoptimized 属性就可以了。
