---
"type:": fleet-note
"title:": 20250801143609-新增taskRunTemplate报错解决方案
id:: 20250801143623  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-08-01T14:36:23  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
错误信息：

```shell
admission webhook "webhook.pipeline.tekton.dev" denied the request: mutation failed: cannot decode incoming new object: json: unknown field
```

出现这个的原因可能是因为引入的版本不对：
```shell
tekton.dev/v1beta1 --> tekton.dev/v1
```
# Reference