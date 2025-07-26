---
"type:": fleet-note
"title:": 20250725155625-Tekton Knative Permission Issue
"id:": 20250725155633
"created:": 2025-07-25T15:56:33
url: 
tags:
  - fleet-note
  - tekton
"processed:": false
"archived:": false
---
```shell

ERROR: cannont deploy the function: knative deployer failed to get the Knative Service: services.serving.knative.dev "func-test" is forbidden: User "system:serviceaccount:ronghuanz:default" cannot get resource "services" in API group "serving.knative.dev" in the namespace "ronghuanz"

```


Solve:

```shell
kubectl create rolebinding "ronghuanz:knative-serving-namespaced-admin:default" --clusterrole=knative-serving-namespaced-admin --serviceaccount="ronghuanz:default" -n ronghuanz

kubectl create rolebinding "ronghuanz:knative-serving-namespaced-admin:ronghuanz" --clusterrole=knative-serving-namespaced-admin --serviceaccount="ronghuanz:ronghuanz" -n ronghuanz
```

[func/hack/allocate.sh at d04ff0a3788074d7a7d5c0887852bd4b7a86b435 · knative/func · GitHub](https://github.com/knative/func/blob/d04ff0a3788074d7a7d5c0887852bd4b7a86b435/hack/allocate.sh#L413)

# Reference