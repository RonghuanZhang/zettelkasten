---
"type:": fleet-note
"title:": 20250513145138-GKE Push the image manually
"id:": 20250513145151
"created:": 2025-05-13T14:51:51
url: 
tags:
  - fleet-note
  - gcp/artifact-repository
"processed:": false
"archived:": false
---

# Clone the Google samples repo

```shell
git clone https://github.com/GoogleCloudPlatform/kubernetes-engine-samples
cd kubernetes-engine-samples/quickstarts/hello-app
```

# Build the image

```shell

# Artifact repository path
us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research

docker build -t us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app:1.0.0
```

# Config the Docker to push

```shell
# config the docker
gcloud auth configure-docker us-central1-docker.pkg.dev

# Output
Adding credentials for: us-central1-docker.pkg.dev
After update, the following will be written to your Docker config file located at [/Users/erpang/.docker/config.json]:
 {
  "credHelpers": {
    "us-central1-docker.pkg.dev": "gcloud"
  }
}

Do you want to continue (Y/n)?  y

Docker configuration file updated.
```

# Push the image to Google Artifact Repository

```shell
docker push us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app:1.0.0

# Output
The push refers to repository [us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app]
5096926e84cb: Pushed 
f702e48e7f92: Pushed 
dc9b510a903b: Pushed 
585c77542105: Pushed 
2388d21e8e2b: Pushed 
c048279a7d9f: Pushed 
1a73b54f556b: Pushed 
2a92d6ac9e4f: Pushed 
bbb6cacb8c82: Pushed 
ac805962e479: Pushed 
af5aa97ebe6c: Pushed 
4d049f83d9cf: Pushed 
9ed498e122b2: Pushed 
577c8ee06f39: Pushed 
c656415fb81c: Pushed 
1.0.0: digest: sha256:ca3dab33c380a5409071f7c13c8b8b2ddbd5060d904d2693da4d91d4af73d232 size: 3445
```


# Run locally

```shell
docker run --rm -p 8080:8080 us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app:1.0.0
```

# Reference
* [[20250513144411-GKE Artifact Repository]]
* [Deploy a Docker containerized web app to GKE  \|  Kubernetes Engine  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/tutorials/hello-app)