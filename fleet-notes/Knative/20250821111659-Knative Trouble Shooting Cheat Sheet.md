---
"type:": fleet-note
"title:": 20250821111659-Knative Trouble Shooting Cheat Sheet
"id:": 20250821111738
"created:": 2025-08-21T11:17:38
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

## 查看 Enovy 配置

```shell
kubectl -n kourier-system port-forward deploy/3scale-kourier-gateway 19000:9901
```

# Reference