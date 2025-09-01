---
"type:": fleet-note
"title:": 20250514165925-Deploy the Knative Service
"id:": 20250514165949
"created:": 2025-05-14T16:59:49
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

# Image

```shell
us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:0e7d83214f46dd3ab0e4f8a330c5bb498e55b9427a66d2b053f6eda768b4be65
```

# By yaml

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: knative-service-hello-go
spec:
  template:
    spec:
      containers:
        - image: us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:0e7d83214f46dd3ab0e4f8a330c5bb498e55b9427a66d2b053f6eda768b4be65
          ports:
            - containerPort: 8080
```

```shell
kubectl apply -f ./XXX.yaml
```

The knative service will be deployed using the default namespace. (TODO: How do I change it ?)

Verify the deployment result:
```shell
kubectl get ksvc
```

The output:
```shell
erpang@erpangdeMacBook-Pro gke-knative-install % kubectl get ksvc
NAME                       URL                                               LATESTCREATED                    LATESTREADY                      READY   REASON
knative-service-hello-go   http://knative-service-hello-go.default.rhz.com   knative-service-hello-go-00001   knative-service-hello-go-00001   True  
```

```shell
34.10.182.94

curl -x http://127.0.0.1:7890 -H "Host: knative-service-hello-go.default.example.com" http://34.10.182.94

```


# Reference
* [Install Serving with YAML - Knative](https://knative.dev/docs/install/yaml-install/serving/install-serving-with-yaml/#configure-dns)