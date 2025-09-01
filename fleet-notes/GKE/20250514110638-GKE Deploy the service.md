---
"type:": fleet-note
"title:": 20250514110638-GKE Deploy the service
"id:": 20250514110646
"created:": 2025-05-14T11:06:46
url: 
tags:
  - fleet-note
  - gcp/gke
"processed:": false
"archived:": false
---


# Yaml

```yaml
---
apiVersion: "apps/v1"
kind: "Deployment"
metadata:
  name: "hello-go-1"
  namespace: "default"
  labels:
    app: "hello-go-1"
spec:
  replicas: 3
  selector:
    matchLabels:
      app: "hello-go-1"
  template:
    metadata:
      labels:
        app: "hello-go-1"
    spec:
      containers:
      - name: "rhz-hello-app-sha256-1"
        image: "us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:0e7d83214f46dd3ab0e4f8a330c5bb498e55b9427a66d2b053f6eda768b4be65"
---
apiVersion: "autoscaling/v2"
kind: "HorizontalPodAutoscaler"
metadata:
  name: "hello-go-1-hpa-1uj3"
  namespace: "default"
  labels:
    app: "hello-go-1"
spec:
  scaleTargetRef:
    kind: "Deployment"
    name: "hello-go-1"
    apiVersion: "apps/v1"
  minReplicas: 1
  maxReplicas: 5
  metrics:
  - type: "Resource"
    resource:
      name: "cpu"
      target:
        type: "Utilization"
        averageUtilization: 80
---
apiVersion: "v1"
kind: "Service"
metadata:
  name: "hello-go-1-service"
  namespace: "default"
  labels:
    app: "hello-go-1"
spec:
  ports:
  - protocol: "TCP"
    port: 8081
    targetPort: 8080
  selector:
    app: "hello-go-1"
  type: "LoadBalancer"
  loadBalancerIP: ""
```


# Node arch error

I met the below error:

```shell
# Get pod
erpang@erpangdeMacBook-Pro hello-app % kubectl get pods
NAME                         READY   STATUS             RESTARTS         AGE
hello-go-1-f49496596-db28v   0/1     CrashLoopBackOff   12 (4m13s ago)   40m

# Describe pod
Name:             hello-go-1-f49496596-db28v
Namespace:        default
Priority:         0
Service Account:  default
Node:             gk3-sahara-ai-serverless-platf-pool-2-2f23f19f-t8mb/10.128.0.5
Start Time:       Wed, 14 May 2025 12:42:30 +0800
Labels:           app=hello-go-1
                  app.kubernetes.io/managed-by=cloud-console
                  pod-template-hash=f49496596
Annotations:      <none>
Status:           Running
SeccompProfile:   RuntimeDefault
IP:               10.10.0.77
IPs:
  IP:           10.10.0.77
Controlled By:  ReplicaSet/hello-go-1-f49496596
Containers:
  rhz-hello-app-sha256-1:
    Container ID:   containerd://cf1e759f4ddeeabfa9dd5768cab8f15dd97c6a2d781acc06341aceaf76fc295f
    Image:          us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:ca3dab33c380a5409071f7c13c8b8b2ddbd5060d904d2693da4d91d4af73d232
    Image ID:       us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:ca3dab33c380a5409071f7c13c8b8b2ddbd5060d904d2693da4d91d4af73d232
    Port:           <none>
    Host Port:      <none>
    State:          Waiting
      Reason:       CrashLoopBackOff
    Last State:     Terminated
      Reason:       Error
      Exit Code:    1
      Started:      Wed, 14 May 2025 13:19:05 +0800
      Finished:     Wed, 14 May 2025 13:19:05 +0800
    Ready:          False
    Restart Count:  12
    Limits:
      ephemeral-storage:  1Gi
    Requests:
      cpu:                500m
      ephemeral-storage:  1Gi
      memory:             2Gi
    Environment:          <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-qjkfd (ro)
Conditions:
  Type                        Status
  PodReadyToStartContainers   True 
  Initialized                 True 
  Ready                       False 
  ContainersReady             False 
  PodScheduled                True 
Volumes:
  kube-api-access-qjkfd:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 kubernetes.io/arch=amd64:NoSchedule
                             node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type     Reason     Age                   From                                   Message
  ----     ------     ----                  ----                                   -------
  Normal   Scheduled  41m                   gke.io/optimize-utilization-scheduler  Successfully assigned default/hello-go-1-f49496596-db28v to gk3-sahara-ai-serverless-platf-pool-2-2f23f19f-t8mb
  Normal   Started    25m (x9 over 41m)     kubelet                                Started container rhz-hello-app-sha256-1
  Normal   Created    20m (x10 over 41m)    kubelet                                Created container: rhz-hello-app-sha256-1
  Normal   Pulled     4m45s (x13 over 41m)  kubelet                                Container image "us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app@sha256:ca3dab33c380a5409071f7c13c8b8b2ddbd5060d904d2693da4d91d4af73d232" already present on machine
  Warning  BackOff    74s (x185 over 41m)   kubelet                                Back-off restarting failed container rhz-hello-app-sha256-1 in pod hello-go-1-f49496596-db28v_default(866837fe-3ac4-4b1f-882c-35e0557b0e19)
```

No idea~~~

![image.png](https://images.hnzhrh.com/note/20250514132614159.png)

View the interactive playbook:

![image.png](https://images.hnzhrh.com/note/20250514132648625.png)

I built the image on my Mac. Check the arch:
```shell
erpang@erpangdeMacBook-Pro hello-app % docker inspect us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app:1.0.0 | grep Arch
        "Architecture": "arm64",
```

Set the CPU architecture then rebuild the image:

```shell
docker build --platform linux/amd64 -t us-central1-docker.pkg.dev/hive-451307/sahara-ai-serverless-platform-repository-research/rhz-hello-app:1.0.0 .
```

Push to the GAR.

# Reference
* [Quickstart: Deploy an app in a container image to a GKE cluster  \|  Google Kubernetes Engine (GKE)  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/archive/deploy-app-container-image)
* [Quickstart: Create a cluster and deploy a workload  \|  Google Kubernetes Engine (GKE)  \|  Google Cloud](https://cloud.google.com/kubernetes-engine/docs/quickstarts/create-cluster)
* [[20250513145138-GKE Push the image manually]]