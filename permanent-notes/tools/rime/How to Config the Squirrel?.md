---
"type:": permanent-note
"title:": How to Config the Squirrel?
"id:": 20250802152703
"created:": 2025-08-02T15:27:03
tags:
  - permanent-note
  - tool/input-method
related-context: 
related-notes:
---


## Where is my Rime?

```shell
/Users/erpang/Library/Rime
```

## Update the Rime ice.

```shell
git pull
```

## How to use the custom config?

You can create a file named `squirrel.custom.yaml` to configure it.

```yaml
patch:
  style/candidate_list_layout: linear
```

# Reference
* [RIME \| 中州韻輸入法引擎](https://rime.im/)
* [配置覆写和定制 \| oh-my-rime输入法](https://www.mintimate.cc/zh/guide/configurationOverride.html)
* [GitHub - iDvel/rime-ice: Rime 配置：雾凇拼音 \| 长期维护的简体词库](https://github.com/iDvel/rime-ice)