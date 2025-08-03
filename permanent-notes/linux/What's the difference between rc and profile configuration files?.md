---
"type:": permanent-note
"title:": What's the difference between rc and profile configuration files?
"id:": 20250802143900
"created:": 2025-08-02T14:39:00
tags:
  - permanent-note
related-context: 
related-notes:
---
`Shell` 终端有很多种，比如 `bash`、`sh` 、`zsh` 等，各自的配置文件名称不一样，比如 `~/.bashrc` `~/.bash_profile` `~/.zsh_rc` `~/.zprofile`

## 加载时机

* `rc`, 启动新的 `shell`、`bash` 实例时加载
* `profile`, 用户登录时加载

## 作用范围

* `.XXrc` 和 `.xx_profile`, 当前用户
* `/etc/profile`, 作用于所有用户

## 最佳实践

将用户配置写在 `rc` 中，在 `profile` 加载 `rc`，比如在 `bash_profile` 中

```shell
source ~/.bashrc
```