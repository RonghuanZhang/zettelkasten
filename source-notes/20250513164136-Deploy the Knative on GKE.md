---
"type:": source-note
"title:": 20250513164136-Deploy the Knative on GKE
"id:": 20250513164150
"created:": 2025-05-13T16:41:50
source:
  - AI
url: 
tags:
  - source-note
"processed:": false
"archived:": false
---
以下是在 Google Kubernetes Engine (GKE) 上部署用于测试 Knative 的完整步骤，包括环境准备、集群创建、Knative Serving 安装与配置等关键环节。请确保在操作前已拥有相应的 Google Cloud 权限（如项目拥有者或 Kubernetes 管理员）以及对 `gcloud`、`kubectl` 等工具的基本使用经验。

## 概要

在 GKE 上部署 Knative 大致分为以下几个阶段：

1. **环境与权限准备**：安装并配置 `gcloud` CLI、`kubectl`，启用所需的 Google Cloud API，并设置项目和身份认证。
    
2. **创建 GKE 集群**：选择标准模式或 Autopilot，指定 Kubernetes 版本（≥ 1.21），配置节点规格及网络。
    
3. **启用 Knative Serving 功能**：通过 Cloud Run for Anthos（Knative Serving）将 Knative 功能添加至 GKE Fleet，并在目标集群上应用。
    
4. **安装网络层插件**：部署 Kourier、Istio 或其他网络层，以支撑流量路由和自动伸缩。
    
5. **验证与示例应用**：确认 `knative-serving` 命名空间正常运行，并部署 Hello World 示例服务以检验功能。
    

---

## 1. 环境与权限准备

### 1.1 安装与配置命令行工具

- 安装并初始化 **Google Cloud SDK (`gcloud` CLI)**，版本需 **≥ 310.0**（Knative 私有集群配置要求） ([Google Cloud](https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/setup?utm_source=chatgpt.com "Setting up Knative serving | Google Cloud"))。
    
- 安装 **kubectl CLI**，推荐版本与目标集群 Kubernetes 版本保持一致或至少 **v1.10+** ([Google Cloud](https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/install/on-gcp?utm_source=chatgpt.com "Installing Knative serving on Google Cloud"))。
    
- 登录并设置默认项目与区域：
    
    ```bash
    gcloud auth login
    gcloud config set project YOUR_PROJECT_ID
    gcloud config set compute/region us-central1
    gcloud config set compute/zone us-central1-c
    ```
    

### 1.2 启用必要的 Google Cloud API

- **Kubernetes Engine API**
    
- **Compute Engine API**
    
- **Cloud Run API**（用于 Anthos 上的 Knative Serving）
    

````bash
gcloud services enable container.googleapis.com compute.googleapis.com run.googleapis.com
``` :contentReference[oaicite:2]{index=2}  

---

## 2. 创建 GKE 集群

Knative 推荐使用 GKE **1.21 及以上** 版本（Autopilot 或标准集群均可） :contentReference[oaicite:3]{index=3}。以下示例为标准模式：

```bash
gcloud container clusters create knative-test-cluster \
  --cluster-version=1.28.0 \
  --machine-type=e2-standard-4 \
  --num-nodes=3 \
  --enable-ip-alias \
  --addons=HorizontalPodAutoscaling,HttpLoadBalancing \
  --enable-stackdriver-kubernetes \
  --region us-central1
````

- `--enable-ip-alias`：开启 VPC 原生网络。
    
- `--addons`：启用自动伸缩与 HTTP 负载均衡。
    
- 如需 Autopilot 模式，可使用 `gcloud container clusters create-auto` ([Gist](https://gist.github.com/WilliamDenniss/3cff8658411ee3342c3e35017c22898c?utm_source=chatgpt.com "Installation of Knative on a GKE Autopilot cluster - GitHub Gist"))。
    

---

## 3. 启用 Knative Serving 功能

### 3.1 将 Knative Serving 添加至 Fleet

1. 为项目启用 Cloud Run for Anthos（即 Knative Serving）功能：
    
    ````bash
    gcloud container fleet cloudrun enable --project=YOUR_PROJECT_ID
    ``` :contentReference[oaicite:5]{index=5}  
    ````
    
2. 验证 Feature 状态：
    
    ```bash
    gcloud container fleet features list --project=YOUR_PROJECT_ID
    ```
    
    确认 `appdevexperience` 状态为 `ACTIVE` ([Google Cloud](https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/install/on-gcp "Installing Knative serving on Google Cloud"))。
    

### 3.2 在集群上应用 Knative

对每个目标集群执行：

```bash
gcloud container fleet cloudrun apply \
  --gke-cluster=us-central1-c/knative-test-cluster \
  --project=YOUR_PROJECT_ID
```

该操作会在集群中自动创建 `knative-serving` 命名空间，并部署 Serving 核心组件及 CRD ([Google Cloud](https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/install/on-gcp "Installing Knative serving on Google Cloud"))。

---

## 4. 安装网络层插件

Knative Serving 需要一个 Ingress 实现，推荐使用 **Kourier**（轻量）或 **Istio**（功能强大）：

### 4.1 Kourier

````bash
kubectl apply -f https://github.com/knative/net-kourier/releases/download/knative-v1.8.2/kourier.yaml
kubectl patch configmap/config-network \
  --namespace knative-serving \
  --type merge \
  --patch '{"data":{"ingress-class":"kourier.ingress.networking.knative.dev"}}'
``` :contentReference[oaicite:8]{index=8}  

### 4.2 Istio（可选）
```bash
# 安装 Istio 控制平面
curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.18.2 sh -
cd istio-1.18.2
export PATH=$PWD/bin:$PATH
istioctl install --set profile=demo -y

# 启用 Knative 的 Istio 支持
kubectl label namespace knative-serving \
  istio-injection=enabled --overwrite
``` :contentReference[oaicite:9]{index=9}  

---

## 5. DNS 与域名配置

为 Knative 服务设置域名解析：
1. 若使用测试域 `example.com`，应用默认域配置：
   ```bash
   kubectl apply -f \
     https://github.com/knative/serving/releases/download/knative-v1.8.2/serving-default-domain.yaml
````

2. 若使用自定义域名，需部署 `cert-manager` 并创建 `ClusterIssuer`，然后参考官方文档进行 DNS 验证与映射 ([Google Cloud](https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/install/on-gcp?utm_source=chatgpt.com "Installing Knative serving on Google Cloud"))。
    

---

## 6. 验证与部署示例服务

1. **确认各组件就绪**：
    
    ```bash
    kubectl get pods -n knative-serving
    ```
    
    所有 `serving-*` Pod 应为 `Running` 状态。
    
2. **部署 Hello World 示例**：  
    创建 `service.yaml`：
    
    ```yaml
    apiVersion: serving.knative.dev/v1
    kind: Service
    metadata:
      name: helloworld-go
    spec:
      template:
        spec:
          containers:
            - image: gcr.io/knative-samples/helloworld-go
              env:
                - name: TARGET
                  value: "Knative on GKE"
    ```
    
    应用并查看状态：
    
    ```bash
    kubectl apply -f service.yaml
    kubectl get ksvc helloworld-go
    ```
    
    获取 URL 并访问：
    
    ```bash
    kubectl get route helloworld-go \
      --output=custom-columns=URL:.status.url
    ```
    

---

完成以上步骤后，您的 GKE 集群即已启用 Knative Serving 并准备好接收 Serverless 服务部署。您可进一步探索 Knative Eventing、自动伸缩与流量分割等高级功能。

# Reference