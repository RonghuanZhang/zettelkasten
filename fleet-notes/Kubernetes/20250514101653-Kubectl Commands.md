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


# Cluster & Context

```shell
# Get the contexts
kubectl config get-contexts

# Swich the context
kubectl config use-context <context>
```

# RBAC

```shell
kubectl auth can-i create services.serving.knative.dev -n ronghzhang3 --as=ronghzhang3
```

```shell
kubectl create rolebinding "${SA}-${ROLE}" \  
--role=${ROLE} \  
--serviceaccount="${NAMESPACE}:${SA}" \  
-n "${NAMESPACE}"

kubectl create rolebinding ronghzhang3-knative-serving-admin \
--role=knative-serving-admin \
--serviceaccount=ronghzhang3:ronghzhang3 \
-n ronghzhang3
```


# 日志

```shell
kubectl logs <pod-name> -c <container-name> -n <namespace>
logs -f
logs --tail
```

# 端口转发

```shell
kubectl port-forward service/hive-orchestration 8000:8080 -n hive

本机端口:service端口
```

# Reference