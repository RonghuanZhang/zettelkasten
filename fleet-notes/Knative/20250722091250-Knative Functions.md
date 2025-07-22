---
"type:": fleet-note
"title:": 20250722091250-Knative Functions
id:: 20250722091256  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-07-22T09:12:56  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---

## func CLI

```shell
# Init the Golang function
func create -l go func-test

func build --registry us-central1-docker.pkg.dev/hive-451307/hive --verbose

func deploy -i us-central1-docker.pkg.dev/hive-451307/hive/func-test@sha256:3aae438ae5f8f131ab88851d9ae8290ed4406fce15e5ba0ea111bc689ef342f4 -n ronghuanz -v --push=false

func deploy -n ronghuanz -v
```


# Tekton 

以下为你系统性设计 **GKE 多租户环境下，使用 Tekton Pipelines 实现：私有 Git 拉取 ➔ Buildpacks 构建 ➔ GCR 推送 ➔ Knative Function 部署** 的最佳实践流程与详细配置思路。

---

## 🏗 **🔶 1. 流程架构总览**

```
Developer Git Push
      |
      v
Tekton PipelineRun
  ├── Task: git-clone (私有 Git repo)
  ├── Task: buildpack-build (pack build ➔ image)
  ├── Task: push-image (GCR push)
  └── Task: deploy-knative (kn service update/create)
```

---

## ⚙️ **🔶 2. 多租户设计要点**

| 维度                      | 设计原则                                                  |
| ----------------------- | ----------------------------------------------------- |
| **命名空间隔离**              | 每租户独立 namespace（例如 team-a, team-b）                    |
| **ServiceAccount 最小权限** | 每 namespace 配置专属 ServiceAccount                       |
| **Secret 隔离**           | Git SSH Secret、GCR Pull/Push Secret 均只存在于租户 namespace |
| **资源配额**                | 每 namespace 配置 ResourceQuota，防止资源争用                   |
| **Tekton Pipelines**    | 安装在 cluster-level，但 PipelineRun 在各 namespace 执行       |

---

## 🔧 **🔶 3. 核心配置模板**

以下为 **逐步配置方案**，以 GKE + GCR + GitHub 私有仓库为例。

---

### 📝 **Step 1. 创建 Git SSH Secret (每租户)**

```bash
kubectl create secret generic github-ssh-secret \
  --type=kubernetes.io/ssh-auth \
  --from-file=ssh-privatekey=<path-to-id_rsa> \
  -n team-a
```

---

### 📝 **Step 2. 创建 GCR Push Secret (每租户)**

```bash
kubectl create secret docker-registry gcr-json-key \
  --docker-server=gcr.io \
  --docker-username=_json_key \
  --docker-password="$(cat keyfile.json)" \
  --docker-email=<email> \
  -n team-a
```

---

### 📝 **Step 3. 创建 ServiceAccount (每租户)**

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-sa
  namespace: team-a
secrets:
  - name: github-ssh-secret
  - name: gcr-json-key
imagePullSecrets:
  - name: gcr-json-key
```

---

### 📝 **Step 4. Pipeline 定义**

```yaml
apiVersion: tekton.dev/v1beta1
kind: Pipeline
metadata:
  name: build-and-deploy-function
  namespace: team-a
spec:
  params:
    - name: git-url
    - name: image-url
    - name: function-name
  tasks:

    - name: clone-repo
      taskRef:
        name: git-clone
      params:
        - name: url
          value: $(params.git-url)
        - name: revision
          value: main
        - name: sshSecretName
          value: github-ssh-secret
      workspaces:
        - name: output
          workspace: shared-workspace

    - name: buildpack-build
      runAfter: [clone-repo]
      taskSpec:
        steps:
          - name: build
            image: buildpacksio/pack
            script: |
              cd $(workspaces.shared-workspace.path)
              pack build $(params.image-url) --builder paketobuildpacks/builder:base --publish
      workspaces:
        - name: shared-workspace
          workspace: shared-workspace

    - name: deploy-knative
      runAfter: [buildpack-build]
      taskSpec:
        steps:
          - name: deploy
            image: ghcr.io/boson-project/func
            script: |
              func deploy --image $(params.image-url) --name $(params.function-name) --namespace team-a

  workspaces:
    - name: shared-workspace
```

---

### 📝 **Step 5. PipelineRun 示例**

```yaml
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: build-and-deploy-run
  namespace: team-a
spec:
  pipelineRef:
    name: build-and-deploy-function
  params:
    - name: git-url
      value: git@github.com:myorg/my-private-repo.git
    - name: image-url
      value: gcr.io/my-project/my-function:latest
    - name: function-name
      value: my-function
  workspaces:
    - name: shared-workspace
      volumeClaimTemplate:
        metadata:
          name: workspace-pvc
        spec:
          accessModes: ["ReadWriteOnce"]
          resources:
            requests:
              storage: 1Gi
  serviceAccountName: pipeline-sa
```

---

## ✅ **🔶 6. 关键安全与治理建议**

1. **ClusterRole 限制**

   * Tekton controller 需要 cluster-level 权限
   * 但 TaskRun/ PipelineRun 执行 SA 仅授予 namespace scope 权限

2. **GCR Push 权限最小化**

   * keyfile.json 仅授予写入目标 repo scope

3. **Secrets namespace 隔离**

   * 不跨 namespace 共享 Git/GCR secret

4. **ResourceQuota & LimitRange**

   * 每 namespace 配置 CPU, Memory 上限，避免 noisy neighbor

5. **NetworkPolicy**

   * 限制 pipeline pod 的外部访问范围

---

## ✨ **🔶 7. 总结优势**

✅ **GitOps 模式**：所有配置和函数代码均在 Git 中维护
✅ **多租户安全**：命名空间隔离 + SA 最小权限
✅ **Buildpacks 易维护**：无需管理 Dockerfile
✅ **Knative 部署一致性**：CI/CD 与本地 `func deploy` 体验一致
✅ **与 ArgoCD 集成**：可将 PipelineRun 作为 GitOps manifest 管理

---

如果需要，我可以为你输出：

* 完整 YAML 文件分层组织方案
* Secret、ServiceAccount、Pipeline、PipelineRun 全套可部署模板
* 结合 ArgoCD、Vault 管理 Git/GCR 凭据的最佳实践方案

告诉我你计划落地的目标时间和 CI/CD 规范要求，我将帮助你形成 **面向生产的 Tekton + Knative Function GitOps 流水线架构方案**。

# Tekton + Knative

以下为 **Knative + Tekton 多租户最佳实践方案**，针对你的 GKE + Function 平台需求，结合企业级落地经验整理，帮助你安全、高效、可扩展地运行。

---

## 🏗️ **一、目标与挑战**

✅ 同一集群多团队共用
✅ 避免不同团队资源冲突
✅ 保证镜像、Secrets、Ingress 隔离
✅ 提供标准化 CI/CD 工作流
✅ 易于 GitOps 管理与自动化

---

## 🔧 **二、核心设计原则**

1. **Namespace 隔离**

   * 每个团队/租户使用独立 namespace
   * Tekton Pipeline、Task、Secret、Knative Service 均在对应 namespace 中运行

2. **RBAC 授权最小化**

   * 每个 namespace 创建 team-specific ServiceAccount
   * 绑定对应 namespace 内的 Role

3. **镜像仓库分隔**

   * 为每个团队分配独立镜像 registry 路径
   * image push 权限与 namespace 绑定

4. **Tekton Workspace PVC 隔离**

   * workspace PVC 设置 accessMode: ReadWriteOnce
   * 使用 dynamic provisioning（StorageClass）避免跨租户共享

5. **Ingress 与域名管理**

   * 为每个 namespace 配置子域名或 path-based ingress
   * e.g. team-a.example.com, team-b.example.com

6. **Pipeline 与 Task 模块化**

   * 公共 task (git-clone, buildpack-build, deploy) 定义在 shared namespace
   * Pipeline 定义在 team namespace 并引用 shared task

---

## ✨ **三、最佳实践架构**

```plaintext
[GKE Cluster]
│
├── [Namespaces]
│    ├── team-a
│    │    ├── Tekton Pipelines (Pipeline, PipelineRun)
│    │    ├── Knative Services (Function)
│    │    ├── ServiceAccount + RoleBinding
│    │    ├── Secrets (Git, Registry)
│    │    └── PVC (workspace)
│    │
│    ├── team-b
│    │    └── ...
│    │
│    └── tekton-shared
│         ├── Tasks (git-clone, buildpack-build, deploy)
│         └── ClusterRoleBinding for shared usage
│
└── [Ingress]
     ├── *.example.com route to Knative Gateway
     └── TLS managed by cert-manager
```

---

## 🔐 **四、RBAC 配置**

✔️ 每个 team namespace 创建：

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: pipeline-sa
  namespace: team-a
---
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: pipeline-role
  namespace: team-a
rules:
  - apiGroups: ["tekton.dev"]
    resources: ["pipelineruns", "taskruns"]
    verbs: ["get", "list", "create", "delete"]
  - apiGroups: ["serving.knative.dev"]
    resources: ["services"]
    verbs: ["get", "list", "create", "update", "delete"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pipeline-rolebinding
  namespace: team-a
subjects:
  - kind: ServiceAccount
    name: pipeline-sa
roleRef:
  kind: Role
  name: pipeline-role
  apiGroup: rbac.authorization.k8s.io
```

---

## 🗃️ **五、Secrets 管理**

* Git SSH Key Secret
* Registry Push Credentials

⚠️ 注意：

* secret scope 仅限 team namespace
* PipelineRun 通过 `serviceAccountName` 使用 secrets

---

## 🏗️ **六、Pipeline 设计模式**

✅ **Shared Task + Namespace Pipeline**

* 在 `tekton-shared` 定义 task：

  * git-clone
  * buildpack-build
  * deploy-knative

* 在 team namespace 定义 pipeline，引用 shared task：

  ```yaml
  taskRef:
    name: git-clone
    namespace: tekton-shared
  ```

---

## 🔄 **七、Knative Serving 配置**

✔️ 每个 team namespace 配置：

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: func-hello
  namespace: team-a
spec:
  template:
    spec:
      serviceAccountName: pipeline-sa
```

✔️ Knative ingress 使用 wildcard DNS + cert-manager 为每个 team 动态生成 TLS。

---

## 🔭 **八、资源与配额管理**

为防止过度消耗资源，每个 namespace 设置：

* ResourceQuota
* LimitRange

示例：

```yaml
apiVersion: v1
kind: ResourceQuota
metadata:
  name: team-a-quota
  namespace: team-a
spec:
  hard:
    requests.cpu: "4"
    requests.memory: 8Gi
    limits.cpu: "8"
    limits.memory: 16Gi
    persistentvolumeclaims: "5"
```

---

## 💡 **九、CI/CD 触发器**

使用 **Tekton Triggers**：

* 每个 team namespace 部署 EventListener
* Git webhook 指向对应 EventListener route

这样，团队提交代码即可自动触发 pipeline，实现 DevOps 自助。

---

## 📝 **十、总结**

| 目标       | 实践方案                             |
| -------- | -------------------------------- |
| **租户隔离** | namespace + RBAC + secret scope  |
| **任务共享** | shared namespace 存放 task         |
| **镜像管理** | 分团队 registry path                |
| **域名隔离** | wildcard subdomain per namespace |
| **资源保护** | ResourceQuota + LimitRange       |

---

若需要，我可为你生成：

✅ **团队模板 YAML**（Namespace, ServiceAccount, Role, Pipeline, PipelineRun）
✅ **一键部署脚本**
✅ **全局架构图**，便于对内分享方案

请告诉我下一步需求，我将继续协助你的 Function 平台多租户落地推进。


# Reference