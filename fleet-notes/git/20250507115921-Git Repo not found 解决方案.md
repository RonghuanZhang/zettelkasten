---
"type:": fleet-note
"title:": 20250507115921-Git Repo not found 解决方案
"id:": 20250507115930
"created:": 2025-05-07T11:59:30
source:
  - web
  - AI
  - wechat
  - book
  - video
url: 
tags:
  - fleet-note
  - git
"processed:": false
"archived:": false
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