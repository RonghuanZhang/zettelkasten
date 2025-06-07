---
"type:": permanent-note
"title:": Repository not found 的解决办法
"id:": 20250607174829
"created:": 2025-06-07T17:48:29
tags:
  - permanent-note
  - git
related-context: 
related-notes:
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