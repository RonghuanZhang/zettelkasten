---
"type:": fleet-note
"title:": 20250730203548-Gemeni Tekton Docker 认证
id:: 20250730203555  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-07-30T20:35:55  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
# 在GKE中使用Tekton部署Knative Func时配置Docker认证密钥

## 1. 执行摘要

在Google Kubernetes Engine (GKE) 上，为持续集成/持续部署 (CI/CD) 流水线（Tekton）和部署的应用（Knative 函数）安全地向私有容器注册表（特别是 GCP Artifact Registry）进行认证，是核心挑战。用户查询明确指出了对 `dockerconfig-workspace` 和 `tekton-docker-secret` 配置的需求。

该分析表明，解决此问题的最安全和推荐方法是**工作负载身份联邦 (Workload Identity Federation)**。这种机制允许 Kubernetes 服务账号 (KSA) 模拟 Google Cloud 服务账号 (GSA)，为访问 Artifact Registry 提供了一种无密钥且自动轮换的认证方式。这种方法消除了在 Kubernetes Secret 中存储静态、长期凭证的必要性 1。

用户查询明确提到了对 `dockerconfig-workspace` 和 `tekton-docker-secret` 的配置需求，这些是 Kubernetes 中用于管理静态凭证的原生对象。然而，深入分析研究材料（例如 1）强烈推荐使用工作负载身份联邦。这揭示了云原生安全范式的一个根本性转变：即从依赖静态、可能长期存在的凭证（如嵌入在

`dockerconfigjson` Secret 中的服务账号密钥）转向利用由托管身份派生的动态、短期令牌。这种转变对持续集成/持续部署（CI/CD）在GCP上的架构决策至关重要。这种方法通过最大限度地减少敏感、长期密钥的暴露，从根本上缩小了攻击面。它还简化了凭证轮换，因为底层的令牌生成和过期由GCP自动处理，从而降低了密钥管理的运营开销，并提高了整体安全态势。

虽然工作负载身份联邦是首选，但传统的 `kubernetes.io/dockerconfigjson` Secret（通常由 GCP 服务账号密钥生成）仍然是一种可行的替代方案，尤其适用于 GKE 之外的环境或特定的遗留集成。然而，这些方法引入了手动密钥管理和更高的安全风险。

本报告将详细阐述这两种认证策略，强调工作负载身份联邦作为最佳实践，并提供 Tekton 流水线（使用 Kaniko 进行构建）和 Knative 函数部署的逐步配置指南。

## 2. Tekton与Knative在GKE上的CI/CD简介

### Tekton for 云原生 CI/CD

Tekton 是一个开源框架，专为开发灵活且可扩展的持续集成和持续部署 (CI/CD) 系统而设计 4。它利用 Kubernetes 自定义资源 (CRs) 将 CI/CD 流水线定义为代码，从而支持 GitOps 原则，实现自动化和版本控制的构建过程 4。

Tekton 的核心组件包括：

- **Pipelines**：定义一系列按序执行的 `Tasks` 5。
    
- **Tasks**：可重用的工作单元，通常是执行特定步骤的容器镜像 4。
    
- **Workspaces**：提供共享存储，用于在 `Tasks` 之间传递输入、输出和凭证 6。
    

### Knative for 无服务器函数部署

Knative Serving 是一个 Kubernetes 原生解决方案，用于部署和管理无服务器工作负载，包括函数。它抽象了复杂的网络和扩缩问题 7。Knative 实现了容器镜像的快速部署，处理自动扩缩（包括缩容到零）、流量管理和版本控制 8。

### GKE 和 GCP Artifact Registry 在生态系统中的作用

Google Kubernetes Engine (GKE) 提供了一个强大、托管的 Kubernetes 环境，作为运行 Tekton 流水线和 Knative 服务的基础。GCP Artifact Registry 是 Google Cloud 推荐的通用包管理器，用于存储各种类型的制品，包括 Docker 容器镜像。它是 Container Registry 的继任者，后者正在被弃用 7。Artifact Registry 与 Google Cloud 的 IAM 深度集成，提供强大的访问控制能力 9。

Tekton 使用 Kubernetes 自定义资源定义其 CI/CD 工作流程 4。Knative 类似地直接在 Kubernetes 上部署和管理容器化应用程序 8。这两个框架都高度依赖标准的 Kubernetes 对象，如

`ServiceAccounts` 和 `Secrets` 来实现身份和认证 8。这种对 Kubernetes 原生 API 及其核心对象的持续依赖，突显了它们作为事实上的通用集成层的重要性。用户关于

`dockerconfig-workspace` 和 `tekton-docker-secret` 的具体问题，直接指向了这些 Kubernetes 原语在认证中的应用。这种架构选择，即不同的云原生工具都利用共同的 Kubernetes API，促进了互操作性并减少了供应商锁定。这意味着对 Kubernetes 基本原理的深入理解，对于有效集成和管理 GKE 上复杂的 CI/CD 和部署流水线来说，不仅有益，而且至关重要。这种标准化简化了整体系统设计和维护。

## 3. Kubernetes和Tekton中的Docker认证机制

### `kubernetes.io/dockerconfigjson` Secret 类型

`kubernetes.io/dockerconfigjson` 是 Kubernetes 中专门为存储 Docker 注册表认证凭证而设计的标准 Secret 类型 13。此 Secret 类型中的

`data` 字段预期包含一个名为 `.dockerconfigjson` 的键，其值是一个 Base64 编码的 JSON 字符串。这个 JSON 字符串代表了 Docker `config.json` 文件的内容，该文件通常包含一个或多个 Docker 注册表的认证详细信息 13。

Base64 解码后的 `.dockerconfigjson` 内容应类似于以下结构：

JSON

```
{
  "auths": {
    "https://<your-registry-server>": {
      "username": "...",
      "password": "...",
      "email": "...",
      "auth": "base64encoded(username:password)"
    }
  }
}
```

`kubectl create secret docker-registry` 是推荐的命令行工具，用于创建此类 Secret，它能自动处理 Base64 编码和正确的格式 14。

### Tekton 如何通过 Service Accounts 消费 Docker 凭证

Tekton 的 `Runs`（包括 `TaskRuns` 和 `PipelineRuns`）通过其关联的 Kubernetes `ServiceAccount` 获得对 Kubernetes Secret（包括 Docker 认证 Secret）的访问权限 12。Tekton 明确支持

`kubernetes.io/dockerconfigjson` 类型的 Secret 12。

为了使 Tekton 能够使用这些凭证，`kubernetes.io/dockerconfigjson` Secret（例如 `tekton-docker-secret`）必须在 `TaskRun` 或 `PipelineRun` 所使用的 `ServiceAccount` 的 `secrets` 字段中被引用 12。Tekton 的内部机制会在执行任何命令之前，自动将这些关联的 Secret 转换并注入到

`Step` 容器内的 `~/.docker/config.json` 文件中。这使得标准的 Docker 命令（如 `docker push` 或 `docker pull`，如果存在 Docker 守护进程，或像 Kaniko 这样读取此文件的工具）能够无缝认证 12。

Tekton 还支持在 Secret 本身添加一个可选注解（`tekton.dev/docker-0: <registry-url>`）。当多个 Docker Secret 与一个 `ServiceAccount` 关联，或当特定域需要明确的凭证映射时，此注解有助于 Tekton 选择正确的凭证 12。如果同时提供了

`kubernetes.io/*` 和 Tekton 风格的基本认证 Secret，Tekton 会合并这些凭证，其中 Tekton 风格的凭证具有优先权 17。

### `dockerconfig-workspace` 的目的和用途

Tekton `Workspaces` 是一种多功能机制，用于提供共享存储并将各种类型的数据（包括 Secret 中持有的凭证）挂载到 `Task` Pod 中 6。

虽然 Tekton 可以通过 `ServiceAccount` 关联的 `imagePullSecrets` 或带注解的 Docker Secret 自动注入 `~/.docker/config.json` 12，但通过

`Workspace` 显式挂载 Secret（如用户查询中 `dockerconfig-workspace` 所建议）提供了更细粒度的控制。这种方法特别有用，如果 `Task` 中的特定工具期望凭证文件位于非标准路径，或者如果 Secret 包含需要由环境变量（如 Kaniko 的 `GOOGLE_APPLICATION_CREDENTIALS`）直接引用的原始服务账号密钥（例如 JSON 文件），而不是格式化为 `dockerconfigjson` 19。需要注意的是，通过 Workspaces 挂载的

`Secret` 卷源始终是只读的 6。该 Secret 必须在

`TaskRun` 提交之前存在 6。

研究表明，Tekton 支持两种主要方式来提供 Docker 凭证：一是当 `kubernetes.io/dockerconfigjson` Secret 链接到 `ServiceAccount` 时，自动注入 `~/.docker/config.json` 文件 12；二是将 Secret 显式挂载为

`Workspace` 6。用户查询中提及

`dockerconfig-workspace` 指向了后一种方式。这种双重机制表明，虽然 Tekton 提供了一种便捷的、符合规范的方式来处理 `dockerconfigjson` Secret，但对于特定用例或工具（例如 Kaniko 可能期望由 `GOOGLE_APPLICATION_CREDENTIALS` 指向的原始服务账号密钥，而不是直接的 `~/.docker/config.json`），直接卷挂载是可用的。这种灵活性意味着用户必须理解 Tekton 认证处理的细微差别。对于一般的 Docker 操作（拉取/推送），`ServiceAccount` 链接的 Secret 通常更简单且足够。然而，对于专用工具或场景（如 Kaniko 利用 GCP 原生认证或原始服务账号密钥），通过 Workspace 挂载原始密钥并相应配置工具是适当的模式。

## 4. GCP Artifact Registry的GKE认证方法

### Artifact Registry 认证选项概述

GCP Artifact Registry 支持多种 Docker 客户端认证方法，包括 `gcloud CLI 凭证助手`、`独立凭证助手`、`访问令牌` 和 `服务账号密钥` 10。对于 GKE 工作负载，最安全和推荐的方法是工作负载身份联邦，而使用服务账号密钥（通常嵌入在

`dockerconfigjson` Secret 中）则是一种安全性较低的替代方案。

用户查询中提及 GCP Artifact Registry 是否有“固定认证”。研究（10）揭示了多种认证

_方法_（凭证助手、访问令牌、服务账号密钥、工作负载身份联邦）。实际上，并没有单一不变的“固定”方法；相反，存在多种机制，每种机制都有其自身的生命周期和安全配置文件。工作负载身份联邦，虽然是 GKE 的一种“固定”_模式_，但它动态生成短期令牌，这本质上并非传统意义上的固定。这种澄清对于理解现代云安全至关重要。云原生环境中的趋势是采用动态、短期且自动管理的凭证（如工作负载身份联邦或 OAuth2 访问令牌所提供的），这些凭证比静态、长期存在的“固定”凭证更安全。这种动态特性是一种安全优势，而非限制。

### 推荐方法：工作负载身份联邦 (Workload Identity Federation for GKE)

**优势**：工作负载身份联邦是 GKE 应用程序（包括 Tekton 流水线和 Knative 服务）访问 Artifact Registry 等 Google Cloud 服务的最佳实践和最安全方式 1。它允许 Kubernetes 服务账号 (KSA) 模拟 Google Cloud 服务账号 (GSA)，从而消除了在 Kubernetes Secret 中直接存储静态、长期凭证（如服务账号密钥）的需要 2。这显著减少了攻击面并简化了凭证管理。

逐步操作：在 GKE 集群和节点池上启用工作负载身份联邦：

工作负载身份联邦必须首先在 GKE 集群级别启用。现有集群可以进行更新 1。

- **使用 `gcloud CLI`**：
    
    Bash
    
    ```
    gcloud container clusters update CLUSTER_NAME \
      --location=LOCATION \
      --workload-pool=PROJECT_ID.svc.id.goog
    ```
    
    1
    
- **使用 Google Cloud 控制台**：导航到 Google Kubernetes Engine -> 集群 -> 选择您的集群 -> 安全部分 -> 点击“编辑工作负载身份” -> 勾选“启用工作负载身份”复选框 -> 保存更改 1。
    
    在集群上启用后，您可以为现有或新的节点池启用它 1。
    

**逐步操作：将 Kubernetes 服务账号链接到 GCP 服务账号**：

- **1. 创建 Kubernetes 命名空间**：如果您的应用程序尚未拥有命名空间：
    
    Bash
    
    ```
    kubectl create namespace <NAMESPACE>
    ```
    
    1
    
- **2. 创建 Kubernetes 服务账号 (KSA)**：此 KSA 将被您的 Tekton `TaskRun`/`PipelineRun` 或 Knative 服务使用：
    
    Bash
    
    ```
    kubectl create serviceaccount <KSA_NAME> --namespace=<NAMESPACE>
    ```
    
    1
    
- **3. 创建 IAM 服务账号 (GSA)**：此 GSA 将被 KSA 模拟。您可以使用现有账号或创建新账号：
    
    Bash
    
    ```
    gcloud iam service-accounts create <IAM_SA_NAME> \
      --project=<IAM_SA_PROJECT_ID>
    ```
    
    1
    
- **4. 授予 IAM 服务账号必要的角色**：授予 GSA 与 Artifact Registry 交互所需的特定权限。例如，对于推送和拉取：
    
    Bash
    
    ```
    gcloud projects add-iam-policy-binding <IAM_SA_PROJECT_ID> \
      --member "serviceAccount:<IAM_SA_NAME>@<IAM_SA_PROJECT_ID>.iam.gserviceaccount.com" \
      --role "roles/artifactregistry.writer" # 或 roles/artifactregistry.reader 仅用于拉取
    ```
    
    1
    
- **5. 创建 IAM 允许策略以进行模拟**：这一关键步骤授予 KSA 模拟 GSA 的权限：
    
    Bash
    
    ```
    gcloud iam service-accounts add-iam-policy-binding <IAM_SA_NAME>@<IAM_SA_PROJECT_ID>.iam.gserviceaccount.com \
      --role roles/iam.workloadIdentityUser \
      --member "serviceAccount:<IAM_SA_PROJECT_ID>.svc.id.goog"
    ```
    
    1
    
- **6. 注解 Kubernetes 服务账号**：此注解建立了 KSA 与 GSA 之间的链接：
    
    Bash
    
    ```
    kubectl annotate serviceaccount <KSA_NAME> \
      --namespace <NAMESPACE> \
      iam.gke.io/gcp-service-account=<IAM_SA_NAME>@<IAM_SA_PROJECT_ID>.iam.gserviceaccount.com
    ```
    
    1
    

授予 Artifact Registry 必要 IAM 角色（推送/拉取）：

研究材料 11 详细说明了 Artifact Registry 的各种 IAM 角色。21 和 21 明确强调了最小权限原则。这意味着简单地授予

`Storage Admin`（19）给服务账号以进行所有操作通常是权限过高的。相反，关键是仅授予拉取所需的

`Artifact Registry Reader` 角色和推送所需的 `Artifact Registry Writer` 角色 11，并且理想情况下，这些权限应限于仓库级别，而非项目范围 10。这在选择工作负载身份联邦的基础上，进一步深化了安全态势。即使在使用工作负载身份联邦的情况下，权限过高的服务账号也可能带来显著的安全风险。遵循最小权限原则是构建健壮云安全体系的基石。

以下表格详细列出了 Artifact Registry 的常见操作及其所需的 IAM 角色：

**表1：Artifact Registry 常见操作的 IAM 角色**

|操作|所需 IAM 角色|描述|上下文（推荐范围）|相关研究材料|
|---|---|---|---|---|
|拉取镜像|`roles/artifactregistry.reader`|查看和获取制品，查看仓库元数据。|仓库或项目|11|
|推送镜像|`roles/artifactregistry.writer`|读取和写入制品。|仓库或项目|11|
|删除镜像/制品|`roles/artifactregistry.repoAdmin`|读取、写入和删除制品。|仓库或项目|21|
|管理仓库|`roles/artifactregistry.admin`|创建和管理仓库及制品。|项目|21|
|Kaniko 推送（一般）|`roles/artifactregistry.writer`|足以将镜像推送到现有 Artifact Registry 仓库。|仓库或项目|11|
|Kaniko 推送（更广泛）|`roles/storage.admin`（粒度较粗，慎用）|授予广泛的存储权限，可能允许在推送时创建仓库。|项目|19|

### 替代方案：使用服务账号密钥作为 `dockerconfigjson` Secret

**何时考虑**：此方法涉及生成一个长期存在的 GCP 服务账号密钥，并将其内容嵌入到 `kubernetes.io/dockerconfigjson` 类型的 Kubernetes Secret 中。由于密钥的静态性质和需要手动轮换，这种方法不如工作负载身份联邦安全，但可在工作负载身份联邦不可行或特定遗留集成环境中使用 10。

**局限性**：静态密钥如果泄露会带来更高的安全风险。它们需要手动轮换，并且删除服务账号密钥并不会立即撤销基于该密钥颁发的短期凭证；必须禁用或删除服务账号本身才能完全撤销 10。

**逐步操作：生成 GCP 服务账号密钥**：

- 导航到 Google Cloud 控制台（IAM 和管理 -> 服务账号）。
    
- 选择或创建一个服务账号。
    
- 在“密钥”选项卡下，点击“添加密钥” -> “创建新密钥” -> “JSON” -> “创建”。这将下载一个 JSON 密钥文件到您的机器 10。
    
- 确保此服务账号拥有必要的 Artifact Registry 角色（例如，`Artifact Registry Writer`） 10。
    

**逐步操作：从服务账号密钥创建 `kubernetes.io/dockerconfigjson` Secret**：

- `docker login` 命令可用于在本地生成必要的 `~/.docker/config.json` 条目。使用 `_json_key` 作为用户名，并将您的 JSON 密钥文件内容作为密码 10。对于 Artifact Registry，如果密钥文件内容已进行 Base64 编码，则使用
    
    `_json_key_base64` 作为用户名 10。
    
- **`docker login` 示例**：
    
    Bash
    
    ```
    # 将 <LOCATION>-docker.pkg.dev 替换为您的 Artifact Registry 主机名（例如，us-central1-docker.pkg.dev）
    # 将 ~/path/to/sa-key.json 替换为您的 JSON 密钥文件路径
    docker login -u _json_key -p "$(cat ~/path/to/sa-key.json)" https://<LOCATION>-docker.pkg.dev
    ```
    
    10
    
- 登录后，`~/.docker/config.json` 文件将包含您的 Artifact Registry 条目。提取相关的 JSON 结构（特别是 `auths` 对象或您的注册表条目）并进行 Base64 编码 13。
    
- **或者，直接使用 `kubectl create secret`**：这是从服务账号密钥创建 Secret 的更简单和推荐方法，无需手动 Base64 编码：
    
    Bash
    
    ```
    kubectl create secret docker-registry tekton-docker-secret \
      --docker-server="https://<LOCATION>-docker.pkg.dev" \
      --docker-username="_json_key" \
      --docker-password="$(cat ~/path/to/sa-key.json | tr -d '\n')" \
      --docker-email="no-email@example.com" # 邮箱通常是可选/占位符
    ```
    
    14 注意
    
    `tr -d '\n'` 用于去除 JSON 密钥中的换行符 15。
    

以下表格对两种认证方法进行了比较，以帮助决策：

**表2：GKE 中 Artifact Registry 认证方法的比较**

|特性/标准|工作负载身份联邦|服务账号密钥（作为 `kubernetes.io/dockerconfigjson` Secret）|
|---|---|---|
|**安全性**|高：无密钥，使用临时令牌，Kubernetes Secret 中不存储静态凭证。遵循最小权限原则。|中低：静态、长期密钥存储在 Kubernetes Secret 中。如果泄露，风险较高。|
|**凭证管理**|自动：凭证由 GCP 管理，令牌生成被抽象化。|手动：需要显式生成密钥、下载和 Base64 编码。|
|**密钥轮换**|自动：令牌是短期有效的，由 GCP 自动轮换。|手动：需要手动轮换密钥并更新 Kubernetes Secret。删除并不能立即撤销旧令牌 24。|
|**易用性**|初始设置较复杂（集群/节点池启用，KSA-GSA 链接），但后续在流水线中持续使用更简单。|初始 Secret 创建较简单，但持续管理（轮换、安全性）负担较大。|
|**GKE 集成**|GKE 工作负载的原生和惯用方式，利用 GKE 元数据服务器。|标准 Kubernetes Secret 机制，但与 GCP 身份生态系统集成度较低。|
|**推荐用途**|所有新的基于 GKE 的 CI/CD 和应用程序部署。|非 GKE 环境、遗留系统或无法利用工作负载身份联邦的特定工具。|
|**相关研究材料**|1|10|

## 5. 配置Tekton进行Artifact Registry认证

### 将 Docker Secret 或工作负载身份与 Tekton Service Account 关联

Kubernetes `ServiceAccount` 是 Tekton `TaskRuns` 和 `PipelineRuns` 获取权限和凭证的中心点。

- **对于 `kubernetes.io/dockerconfigjson` Secret**：如果您选择使用静态 `dockerconfigjson` Secret（例如 `tekton-docker-secret`），它必须在 `ServiceAccount` YAML 的 `secrets` 字段中被引用。Tekton 会自动将这些凭证注入到任务 Pod 内的 `~/.docker/config.json` 中 12。
    
    YAML
    
    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tekton-builder-sa
      namespace: <your-tekton-namespace>
    secrets:
      - name: tekton-docker-secret # 您的 kubernetes.io/dockerconfigjson secret 的名称
    ```
    
- **对于工作负载身份联邦**：如果使用工作负载身份联邦，`ServiceAccount` 需要用它将模拟的 GCP 服务账号电子邮件进行注解。此 `ServiceAccount` 将自动从链接的 GSA 获取必要的权限，而无需显式的 `dockerconfigjson` Secret 1。
    
    YAML
    
    ```
    apiVersion: v1
    kind: ServiceAccount
    metadata:
      name: tekton-builder-sa
      namespace: <your-tekton-namespace>
      annotations:
        iam.gke.io/gcp-service-account: <GCP_SA_NAME>@<GCP_PROJECT_ID>.iam.gserviceaccount.com
    ```
    

### 将认证集成到 Tekton Tasks 中（以 Kaniko 为例）

Kaniko (`gcr.io/kaniko-project/executor`) 是在 Kubernetes 集群内构建容器镜像而无需 Docker 守护进程的首选工具 19。

研究表明，Kaniko 的文档提到了使用 Kubernetes Secret 或 `GOOGLE_APPLICATION_CREDENTIALS` 19。然而，多份资料（2）明确指出 Kaniko 可以使用工作负载身份联邦推送到 GCR/Artifact Registry，这意味着它会自动利用 GKE 元数据服务器进行认证 20。这表示如果工作负载身份联邦配置正确，运行 Kaniko 的 Tekton Task

_不需要_通过 Workspace 挂载或链接到 `ServiceAccount` 的显式 `dockerconfigjson` Secret _即可推送到 Artifact Registry_。这显著简化了 Tekton Task 的定义，并减少了流水线中手动 Secret 管理的需求。工作负载身份联邦不仅增强了安全性，还通过抽象化构建过程中的凭证处理，简化了 CI/CD 流水线定义。这使得 Tekton Tasks 更清晰、更简洁、更易于维护，降低了流水线开发人员的认知负担。

- Kaniko 与工作负载身份联邦（推荐）：
    
    当 GKE 集群和 Tekton ServiceAccount 正确配置了工作负载身份联邦时，Kaniko 通过利用 GKE 元数据服务器获取凭证来自动认证到 Artifact Registry 2。这意味着 Kaniko 无需显式的
    
    `dockerconfigjson` Secret 或指向挂载密钥的 `GOOGLE_APPLICATION_CREDENTIALS` 环境变量即可推送到 Artifact Registry。
    
    - **使用 Kaniko 的 Tekton Task Step 示例（工作负载身份联邦）**：
        
        YAML
        
        ```
        # Tekton Task 定义的摘录
        spec:
          params:
            - name: IMAGE
              type: string
              description: 要构建和推送的镜像名称（引用）。
          steps:
            - name: build-and-push
              image: gcr.io/kaniko-project/executor:v1.x.x # 使用稳定的 Kaniko 版本
              command:
                - /kaniko/executor
              args:
                - --dockerfile=Dockerfile
                - --context=$(workspaces.source.path) # 假设 'source' workspace 提供构建上下文
                - --destination=$(params.IMAGE)
                # 如果配置了工作负载身份联邦，则无需显式认证标志
                # 对于调试或特定场景，您可以使用 --verbosity=debug
              workspaces:
                - name: source # 用于提供源代码和 Dockerfile 的 workspace
        ```
        
        26
        

用户查询特别提到了 `dockerconfig-workspace` 和 `tekton-docker-secret`。Tekton 文档 12 明确指出，通过

`secrets` 字段将 `kubernetes.io/dockerconfigjson` Secret 与 `ServiceAccount` 关联，将导致 Tekton 在 Pod 内部自动生成 `~/.docker/config.json`。相反，将其挂载为 Workspace 6 则提供了原始的 Secret 数据。如果

`tekton-docker-secret` 是 `kubernetes.io/dockerconfigjson` 类型，那么 `ServiceAccount` 上的 `secrets` 字段是 Tekton 自动注入的更惯用方式。如果它是原始的服务账号 JSON 密钥，那么将其挂载为 Workspace 并为 Kaniko 设置 `GOOGLE_APPLICATION_CREDENTIALS` 才是正确的模式。这种区别对于正确实现至关重要。理解 Tekton 用于凭证注入的具体机制对于避免冗余或不正确的配置至关重要。对于用于通用 Docker CLI 用途的 `kubernetes.io/dockerconfigjson` Secret，应依赖 `ServiceAccount` 链接。对于原始服务账号密钥（例如，当不使用工作负载身份联邦时用于 Kaniko），应使用卷挂载和环境变量。这确保了正确的凭证格式呈现给消费工具。

- Kaniko 与服务账号密钥（通过 Workspace 挂载）：
    
    如果未使用工作负载身份联邦，或者特定场景需要直接使用服务账号密钥，Kaniko 可以配置为使用它。服务账号密钥（JSON 文件）将存储在 Kubernetes Secret（例如 tekton-docker-secret）中，并通过 Workspace 挂载到 Kaniko 容器中。然后，GOOGLE_APPLICATION_CREDENTIALS 环境变量将指向挂载的密钥文件 19。
    
    - **使用 Kaniko 的 Tekton Task Step 示例（挂载服务账号密钥）**：
        
        YAML
        
        ```
        # Tekton Task 定义的摘录
        spec:
          params:
            - name: IMAGE
              type: string
              description: 要构建和推送的镜像名称（引用）。
          steps:
            - name: build-and-push
              image: gcr.io/kaniko-project/executor:v1.x.x
              env:
                - name: GOOGLE_APPLICATION_CREDENTIALS
                  value: /tekton/creds/dockerconfig-workspace/sa-key.json # 挂载密钥文件的路径
              volumeMounts:
                - name: dockerconfig-workspace # 此 workspace 挂载 secret
                  mountPath: /tekton/creds/dockerconfig-workspace
              command:
                - /kaniko/executor
              args:
                - --dockerfile=Dockerfile
                - --context=$(workspaces.source.path)
                - --destination=$(params.IMAGE)
              workspaces:
                - name: source
                - name: dockerconfig-workspace # workspace 定义
                  secret:
                    secretName: tekton-docker-secret # 此 secret 包含 SA 密钥（例如 sa-key.json）
        ```
        
        6
        

### Tekton TaskRun/PipelineRun 示例（使用工作负载身份联邦）

要使用配置好的 `ServiceAccount`（通过工作负载身份联邦链接到 GCP 服务账号）执行 Tekton `Task` 或 `Pipeline`，只需在 `TaskRun` 或 `PipelineRun` 定义中指定 `serviceAccountName`：

YAML

```
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-push-knative-func-run
  namespace: <your-tekton-namespace>
spec:
  serviceAccountName: tekton-builder-sa # 引用上面配置的 ServiceAccount
  pipelineRef:
    name: build-push-knative-func-pipeline # 您的 Tekton Pipeline 名称
  workspaces:
    - name: source # 源代码的示例 workspace（例如 PVC）
      volumeClaimTemplate:
        spec:
          accessModes:
          resources:
            requests:
              storage: 1Gi
  params:
    - name: IMAGE_URL
      value: <LOCATION>-docker.pkg.dev/<PROJECT_ID>/<REPO_NAME>/my-knative-func
    - name: IMAGE_TAG
      value: latest
```

12

## 6. 使用Artifact Registry镜像部署Knative函数

### Knative 的镜像拉取机制和 `imagePullSecrets`

当部署 Knative Service 时，底层的 Kubernetes 集群需要从注册表拉取指定的容器镜像 8。Kubernetes 使用

`imagePullSecrets` 处理私有注册表的认证，这些 Secret 可以直接在 Pod 定义中指定，或者更常见的是在 Pod 所使用的 `ServiceAccount` 上指定 8。默认情况下，如果 Knative Service 定义中未指定自定义

`serviceAccountName`，Knative 服务会使用其各自命名空间中的 `default` `ServiceAccount` 8。为了能够从 Artifact Registry 拉取镜像，Knative Service 所使用的

`ServiceAccount` 必须具有必要的权限。

### 确保 Knative Service Accounts 拥有 Artifact Registry 的适当权限

Tekton（用于 CI/CD 期间构建和推送镜像）和 Knative（用于拉取和部署镜像）都需要对 Artifact Registry 进行认证。研究清楚地表明，工作负载身份联邦可以有效地服务于_这两种_目的（2 适用于 Tekton/Kaniko；3 适用于 Knative/GKE 镜像拉取）。这突显了一个重要的架构优势：可以在 GKE 上的整个 CI/CD 和部署生命周期中应用单一、一致且安全的认证策略。这降低了复杂性，最大限度地减少了配置漂移，并降低了配置错误的潜在风险。将工作负载身份联邦作为 GKE 的基础安全原语，可以简化从源代码到运行应用程序的整个云原生开发工作流程。这种一致性使得整个系统更加健壮、更易于审计，并且更易于大规模管理，这是 DevOps 团队的关键优势。

- 使用工作负载身份联邦（GKE 推荐）：
    
    这是 Knative 在 GKE 上最安全和简化的方法。Knative Service 所使用的 Kubernetes ServiceAccount（例如 default ServiceAccount 或自定义 ServiceAccount）应链接到具有 Artifact Registry Reader (roles/artifactregistry.reader) 权限的 GCP 服务账号 3。当工作负载身份联邦配置正确时，Knative 服务无需在 YAML 定义中显式指定
    
    `imagePullSecrets` 即可从 Artifact Registry 拉取镜像，因为底层 Pod 将自动利用 GKE 元数据服务器进行认证 3。将 KSA 链接到 GSA 的过程与第 4.3.2 节中描述的相同。
    
- 使用 imagePullSecrets（从服务账号密钥）：
    
    如果未使用工作负载身份联邦，则必须创建一个包含 Artifact Registry 凭证的 kubernetes.io/dockerconfigjson Secret（如第 4.4.2 节所述）。然后，此 Secret 必须添加到 Knative 服务将使用的 Kubernetes ServiceAccount 的 imagePullSecrets 列表中 8。
    
    - **示例：将 `imagePullSecrets` 添加到默认 ServiceAccount**：
        
        Bash
        
        ```
        kubectl edit serviceaccount default --namespace <your-knative-namespace>
        ```
        
        在 `imagePullSecrets` 下添加以下内容：
        
        YAML
        
        ```
        apiVersion: v1
        kind: ServiceAccount
        metadata:
          name: default
          namespace: <your-knative-namespace>
        #... 其他字段...
        imagePullSecrets:
          - name: tekton-docker-secret # 您的 kubernetes.io/dockerconfigjson secret 的名称
        ```
        
        8
        
        或者，您可以为 Knative Service 指定一个自定义 ServiceAccount，并将 imagePullSecrets 添加到该自定义 ServiceAccount，或者如果支持，直接在 Knative Service YAML 中指定。
        

### 部署 Knative 服务

使用 `gcloud run deploy` 或应用 Knative Service YAML。确保 `image` URL 指向您的 Artifact Registry 镜像。

- **`gcloud run deploy` 示例**：
    
    Bash
    
    ```
    gcloud run deploy <SERVICE_NAME> \
      --image <LOCATION>-docker.pkg.dev/<PROJECT_ID>/<REPO_NAME>/my-knative-func:latest \
      --cluster=<CLUSTER_NAME> \
      --cluster-location=<CLUSTER_LOCATION> \
      --namespace=<your-knative-namespace> \
      --service-account=default # 或您的自定义 KSA
    ```
    
    8
    

## 7. 总结与最佳实践

### 工作负载身份：首选路径

重申工作负载身份联邦是用于认证 Tekton 流水线和 Knative 函数到 GCP Artifact Registry 的最安全、可扩展和操作高效的方法。其无密钥性质、对临时令牌的依赖以及自动凭证轮换显著增强了安全性并减少了管理开销 2。

### 遵循最小权限原则

始终授予 GCP 服务账号所需的最小 IAM 角色。如果更细粒度的角色（如 `Artifact Registry Writer` 或 `Reader`）足以满足特定操作的需求，则应避免使用 `Storage Admin` 等过于宽泛的角色 10。在可能的情况下，应在仓库级别而非项目范围授予权限，以进一步限制在发生泄露时的影响范围 10。定期审查和审计 IAM 策略。

### 选择正确的凭证策略

对于 GKE 原生工作负载，应优先使用工作负载身份联邦。它简化了 Tekton Task（特别是与 Kaniko 配合使用时）和 Knative 镜像拉取中的凭证管理。仅当工作负载身份联邦不可用时（例如，非 GKE 集群、特定遗留工具），才使用 `kubernetes.io/dockerconfigjson` Secret，并务必勤于密钥轮换和安全存储。

### 常见认证问题排查

本报告涵盖了认证机制（工作负载身份联邦、静态 Secret）和授权（IAM 角色）。故障排除部分明确地结合了这些元素。这表明，要实现真正安全的环境，需要一个整体的视角，而不仅仅关注某个孤立的方面。例如，配置错误的 IAM 角色可能会抵消工作负载身份联邦等安全认证机制的优势，反之亦然。云原生环境中的安全性本质上是多层次的。有效的实施和维护需要理解身份、认证、授权以及流水线中使用的特定工具（Tekton、Kaniko、Knative）之间错综复杂的相互作用。这种集成理解对于构建弹性且安全的系统至关重要。

- **工作负载身份联邦**：验证工作负载身份联邦是否在集群和节点池级别都已启用 1。仔细检查 Kubernetes 服务账号的注解和 IAM 策略绑定以进行模拟 1。确保 GCP 服务账号具有正确的 Artifact Registry IAM 角色 11。
    
- **`dockerconfigjson` Secret**：确认 `kubernetes.io/dockerconfigjson` Secret 格式是否正确（Base64 编码的 JSON） 13。确保 Secret 在 Tekton
    
    `ServiceAccount` 的 `secrets` 字段或 Knative `ServiceAccount` 的 `imagePullSecrets` 中被正确引用 8。
    
- **Kaniko**：如果使用挂载的服务账号密钥，确保 `GOOGLE_APPLICATION_CREDENTIALS` 环境变量正确指向容器内挂载的密钥文件 19。

# Reference