---
"type:": fleet-note
"title:": 20250514101653-Kubectl Commands
"id:": 20250514101659
"created:": 2025-05-14T10:16:59
url: 
tags:
  - fleet-note
  - kubernetes/kubectl
  - cli
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
# 
kubectl logs <pod-name> -c <container-name> --previous
```

# 端口转发

```shell
kubectl port-forward service/hive-orchestration 8000:8080 -n hive

本机端口:service端口
```

# 指定容器

```shell
kubectl logs hive-application-0 -c fix-permissions -n hive
```

# 删除卡住的资源

```shell
kubectl get pvc dumps-hive-application-0 -n hive -o json | jq .metadata.finalizers
```

```shell
kubectl patch pvc dumps-hive-application-0 -n hive -p '{"metadata":{"finalizers":null}}' --type=merge

```

# 测试用端口转发

```shell
kubectl port-forward --namespace logging service/log-collector 8089:80

本地访问8089端口即可
```

# 查看 POD 的 service account

```shell
kubectl get pod func-go-01-00001-deployment-55d6798fd6-cnjtl -n ronghuanz -o jsonpath='{.spec.serviceAccountName}'
```

# 查看 POD 的所有 containers

```shell
kubectl get pod hive-func-test-01-pack-git-pipeline-run-lq7xw-build-pod   -n ronghuanz -o jsonpath='{.spec.containers[*].name}'

```

# 查看Deployment 的 Service Account

```shell
kubectl get deployment <deployment> -n <namespace> -o jsonpath='{.spec.template.spec.serviceAccountName}'
```

# 查看 Kubeconfig 的用户

```shell
kubectl config view --minify -o jsonpath='{.contexts[0].context.user}'
```

# 查看用户的权限

```shell
kubectl auth can-i impersonate users

kubectl auth can-i impersonate users --as='system:serviceaccount:hive-orchestration:hive-orchestration'

kubectl auth can-i get pod --as='system:serviceaccount:ronghuanz20:ronghuanz20'
```
# Reference