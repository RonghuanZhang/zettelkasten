---
"type:": fleet-note
"title:": 20250520163533-HTTP API of Knative
"id:": 20250520163550
"created:": 2025-05-20T16:35:50
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

## Step 1: Get the Public End-point of the Current Cluster


## Step 2: Get the token.

```shell
gcloud auth print-access-token
```

## Step 3: Invoke

```shell
curl -k -H "Authorization: Bearer <token>" https://<public-endpoint>/apis/serving.knative.dev/v1/namespaces/<namespace>/services
```

# Reference