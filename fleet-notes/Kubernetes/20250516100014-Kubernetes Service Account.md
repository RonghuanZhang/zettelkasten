---
"type:": fleet-note
"title:": 20250516100014-Kubernetes Service Account
"id:": 20250516100034
"created:": 2025-05-16T10:00:34
url: 
tags:
  - fleet-note
  - kubernetes/service-account
"processed:": false
"archived:": false
---

# Create the service account

```shell
kubectl create sa ronghzhang-sa -n user-ronghzhang
serviceaccount/ronghzhang-sa created

kubectl get sa -n user-ronghzhang
NAME            SECRETS   AGE
default         0         3h55m
ronghzhang-sa   0         10s
```



# Reference
* [About service accounts in GKE  \|  Google Kubernetes Engine (GKE)  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/how-to/service-accounts)
* [Best practices for GKE RBAC  \|  Google Kubernetes Engine (GKE)  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/best-practices/rbac#kubernetes_service_accounts)
* [[20250516155252-Kubernetes v1.25创建ServiceAccount未生成Secret问题 - immaxfang - 博客园]]