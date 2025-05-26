---
"type:": fleet-note
"title:": 20250526214431-K8s API Design Pattern
id:: 20250526214439  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-05-26T21:44:39  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

API格式为:
```shell
/apis/{group}/{version}/{resource}
```

其中version有以下几种形式：
* v1，表示稳定版本，不会修改，可以使用
* v1alpha1，改功能正在试验中，不推荐应用在生产环境
* v1beta1，功能基本可用，希望用户尝试并提供反馈
# Reference
* [2.3 Kubernetes API \| 《Kubernetes设计与实现》](https://renhongcai.gitbook.io/kubernetes/di-er-zhang-kubernetes-ji-chu/2.3-kubernetes_api)