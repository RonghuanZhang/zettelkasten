---
"type:": fleet-note
"title:": 20250514101033-GKE Access the Cluster By Kubectl
"id:": 20250514101047
"created:": 2025-05-14T10:10:47
url: 
tags:
  - fleet-note
  - gcp/gke
"processed:": false
"archived:": false
---


## Step 1: Enable the Kubernetes API to access the project.

Enable **Google Kubernetes Engine API** in the console.

## Step 2: Install kubectl
Skip it if you had it.

```shell
gcloud components install kubectl
kubectl version --client
```

## Step 3: Install the kubectl auth plugin for GCP

```shell
gke-gcloud-auth-plugin --version

# Install if you don't have gthe plugin
gcloud components install gke-gcloud-auth-plugin

```

## Step 4: Update the kubectl configuration to use the plugin.

```shell
gcloud container clusters get-credentials CLUSTER_NAME \
    --location=CONTROL_PLANE_LOCATION

sahara-ai-serverless-platform-research
us-central1

gcloud container clusters get-credentials sahara-ai-serverless-platform-research --location=us-central1
```

## Step 5: Verify the configuration.

```shell
kubectl get namespaces
```

Check the kubectl config
```shell
kubectl config view
cat ~/.kube/config
```

# Reference
* [Install kubectl and configure cluster access  \|  Google Kubernetes Engine (GKE)  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/how-to/cluster-access-for-kubectl)