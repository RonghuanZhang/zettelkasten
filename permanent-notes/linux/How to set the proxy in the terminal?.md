---
"type:": permanent-note
"title:": How to set the proxy in the terminal?
"id:": 20250802142951
"created:": 2025-08-02T14:29:51
tags:
  - permanent-note
  - linux
  - cli
  - proxy
related-context: 
related-notes:
---

将以下内容写入
* `~/.bash_profile`
* `~/.zshrc`, MacOS 默认使用

关于 `rc` 文件和 `profile` 区别可以参考：[[What's the difference between rc and profile configuration files?]]

```shell
# 代理设置
url=http://127.0.0.1:7890

# 如果代理失效的话直接运行 poff 即可断开 proxy
alias proxy_off='unset http_proxy;unset https_proxy'

# 快捷方式打开
alias proxy_on='export http_proxy=$url; export https_proxy=$url'
```