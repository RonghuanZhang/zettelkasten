---
"type:": fleet-note
"title:": 20250604132220-RoleBinding VS ClusterRoleBinding
id:: 20250604132246  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-06-04T13:22:46  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
RoleBinding scope is namespace.

ClusterRoleBinding scope is global.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: bind-knative-core-role-to-ronghzhang
  namespace: user-ronghzhang
subjects:
- kind: ServiceAccount
  name: ronghzhang-sa
  namespace: user-ronghzhang
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: knative-serving-core
```
# Reference