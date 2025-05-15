---
"type:": fleet-note
"title:": 20250515172522-Knative CLI service
"id:": 20250515172539
"created:": 2025-05-15T17:25:39
url: 
tags:
  - fleet-note
  - cloud-native/serverless/knative
  - cli
"processed:": false
"archived:": false
---



```shell

# List the service
kn service list

# Create the service
kn service create rhz-greetings --port 9080 --image us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-greetings-app:1.0.0

# Delete the service
kn service delete <service-name>

```

# Reference