---
"type:": permanent-note
"title:": Docker构建指定平台镜像
"id:": 20250607162637
"created:": 2025-06-07T16:26:37
tags:
  - permanent-note
  - docker
related-context: 
related-notes:
---
在 Mac OS 系统下构建镜像是，总是忘记操作系统架构的问题，导致在 Linux 架构无法使用在 Mac OS 系统下构建的镜像，可以在构建镜像时指定操作系统架构来避免这个问题。

```shell
docker build --platform linux/amd64
```

常见的操作系统架构：
* `linux/amd64`, 64位Linux，x86_64架构，PC和服务器最常见
* `linux/arm64`
* `windows/amd64`