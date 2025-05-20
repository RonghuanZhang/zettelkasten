---
"type:": fleet-note
"title:": 20250514101653-Kubectl Commands
"id:": 20250514101659
"created:": 2025-05-14T10:16:59
url: 
tags:
  - fleet-note
  - kubernetes/kubectl
"processed:": false
"archived:": false
---
# Deployment

## Scale the deployment pods

```shell

# Scale the deployment resource
kubectl scale deployment hello-deployment --replicas=1
```


# Others
![image.png](https://images.hnzhrh.com/note/20250514101702072.png)
![image.png](https://images.hnzhrh.com/note/20250514101739337.png)


# CRD

```shell
# Get and filter the CRDs.
kubectl get crd | grep 'knative'
```


## Account service

```shell
# Get the token for test.
kubectl -n user-ronghzhang create token ronghzhang-sa
```

# Reference