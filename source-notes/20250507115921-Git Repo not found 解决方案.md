---
type:: fleet-note           # 笔记类型，区分闪念/文献/永久等类别
title:: 20250507115921-Git Repo not found 解决方案
id:: 20250507115930  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-05-07T11:59:30  # 创建时间（ISO 格式）
source:
- web
- AI
- wechat
- book
- video
url:
tags: [fleet-note]         # 便于快速筛选闪念笔记
processed:: false         # 是否已处理（由闪念/文献转永久）
archived:: false          # 是否已归档（处理完成后置为 true）
---

 使用 HTTPS 的方式

清空之前的认证
```shell
$ git credential-osxkeychain erase
host=github.com
protocol=https
```

# Reference
* [git问题ERROR: Repository not found.的解决办法而实际上你的远程库是存在并且可用的，这篇文章 - 掘金](https://juejin.cn/post/6844903681104543758)