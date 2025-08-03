---
"type:": fleet-note
"title:": 20250601124756-Install Rime Input Method
"id:": 20250601124807
"created:": 2025-06-01T12:48:07
url: 
tags:
  - fleet-note
  - tool/input-method
"processed:": false
"archived:": false
---

# 如何自定义Squirrel的配置？

路径在 User Library/Rime 下，更新词库：

```shell
git pull
```

新建有个文件：squirrel.custom.yaml

```yaml
patch:
  style/candidate_list_layout: linear
```

# Reference
* [RIME \| 中州韻輸入法引擎](https://rime.im/)
* [配置覆写和定制 \| oh-my-rime输入法](https://www.mintimate.cc/zh/guide/configurationOverride.html)