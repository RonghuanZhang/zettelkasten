---
"type:": fleet-note
"title:": 20250516144300-Knative Resource Isolation Solution
"id:": 20250516144314
"created:": 2025-05-16T14:43:14
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
  - kubernetes
"processed:": false
"archived:": false
---


There are pre-defined ClusterRules of Knative CRDs, such as `knative-serving-core` (install file: knative-serving-core). We can use it directly.

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
  kind: Role
  name: knative-serving-core
```

How to verify？
```shell
kubectl -n user-ronghzhang create token ronghzhang-sa
```

```shell
kubectl --token=<token> get ksvc -n default
```

![image.png](https://images.hnzhrh.com/note/20250516160951298.png)
# Reference
* [[20250516100734-Kubernetes Resource Isolation]]