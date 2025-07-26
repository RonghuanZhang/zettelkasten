---
"type:": fleet-note
"title:": 20250724142712-Tekton Affinity Pending Issue
"id:": 20250724142740
"created:": 2025-07-24T14:27:40
url: 
tags:
  - fleet-note
  - tekton
"processed:": false
"archived:": false
---

```shell
Type: PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace) ClaimName: func-test-pack-git-pipeline-pvc ReadOnly: false kube-api-access-bcfn8: Type: Projected (a volume that contains injected data from multiple sources) TokenExpirationSeconds: 3607 ConfigMapName: kube-root-ca.crt ConfigMapOptional: <nil> DownwardAPI: true QoS Class: BestEffort Node-Selectors: <none> Tolerations: node.kubernetes.io/not-ready:NoExecute op=Exists for 300s node.kubernetes.io/unreachable:NoExecute op=Exists for 300s Events: Type Reason Age From Message ---- ------ ---- ---- ------- Warning FailedScheduling 2m57s default-scheduler 0/3 nodes are available: 1 node(s) didn't match pod affinity rules, 2 node(s) didn't find available persistent volumes to bind. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling. Warning FailedScheduling 2m53s (x2 over 2m55s) default-scheduler 0/3 nodes are available: 1 node(s) didn't match pod affinity rules, 2 node(s) didn't find available persistent volumes to bind. preemption: 0/3 nodes are available: 3 Preemption is not helpful for scheduling. Normal NotTriggerScaleUp 2m56s cluster-autoscaler pod didn't trigger scale-up: 1 node(s) didn't find available persistent volumes to bind
```


这是一个非常好的问题，涉及 **Tekton Pipelines 的架构设计理念与 Kubernetes Volume 使用的底层限制**。

---

### 🎯 **核心原因**

#### ✅ **1. Kubernetes PVC 的挂载限制**

* 大多数 StorageClass（例如 GKE standard、AWS gp2/gp3）提供的 PVC **AccessMode 为 ReadWriteOnce (RWO)**。
* RWO 的含义：PVC **同一时间只能被挂载到一个 node**。
* 如果多个 Pod 使用同一个 RWO PVC ➔ 它们必须运行在同一个 node 上。

---

#### ✅ **2. Tekton Pipelines 的工作目录特性**

Tekton 使用 **Workspace PVC** 实现：

* Git clone 输出 ➔ 供 buildpacks build 使用
* Task 之间通过 PVC 共享 workspace

若没有 Affinity 机制，Pipeline 的多个 TaskRun Pod 可能会被调度到不同 node ➔

🔴 **结果**:
Pod 启动时 Kubernetes 尝试 attach PVC 到新的 node ➔ 由于 PVC 已被另一个 node attach ➔ attach operation 阻塞 ➔ Pod Pending。

---

### ⚠️ **3. Affinity Assistant 的作用**

为了解决上述问题，Tekton Pipelines 引入了：

> **Affinity Assistant Controller**

* 对使用相同 PVC 的 TaskRun Pod，创建一个 **StatefulSet Pod (Affinity Assistant)**。
* 所有 TaskRun Pod 会被调度到与 Affinity Assistant Pod 相同的 node。

这样可以保证：

✅ Pod 一定可以 attach 使用该 PVC
✅ 避免调度到其他 node 导致 Pending

---

### 🔎 **4. 为什么不默认禁用？**

如果禁用 Affinity Assistant，PipelineRun 中不同 TaskRun Pod 会调度到不同 node，使用 RWO PVC 时会：

* 在第一个 Pod attach PVC 后
* 后续 Pod 调度到不同 node 时 attach 失败 ➔ Pod Pending 无法运行。

---

### ⚠️ **5. RWX 存储可以避免该限制**

如果使用 **ReadWriteMany (RWX)** 类型 PVC（例如：

* GKE Filestore CSI
* EKS EFS CSI
* NFS PV

），同一个 PVC 可以被多个 node 的 Pod 同时挂载。

此时 Affinity Assistant 可关闭，Pod 调度不再受 Node 限制 ➔ 提升并行度与可扩展性。

---

### 🎯 **总结：Tekton 为什么需要 Affinity Assistant**

| 原因                          | 解释                                                                |
| --------------------------- | ----------------------------------------------------------------- |
| **K8s RWO PVC 限制**          | PVC 只能 attach 到一个 node，Pipeline 中多个 Task 需要在同一 node 才能访问 PVC      |
| **Pipeline 多 Task 调度**      | Kubernetes scheduler 默认将 Pod 分散到不同 node，提高集群利用率，但这会与 RWO PVC 限制冲突 |
| **Affinity Assistant 解决方案** | 保证 Pod 调度到同一 node，避免 attach 失败，保障 Pipeline 成功执行                   |

---

### ✅ **最佳实践**

1. **生产环境推荐 RWX 存储** ➔ 提升并行度，避免 Affinity Assistant 复杂性
2. **小型集群可使用 Affinity Assistant** ➔ 需关注单节点资源瓶颈
3. **了解 AccessModes 与 Pod Scheduling 的关系** ➔ 设计 Tekton Pipelines 时避免调度失败

---

如果你需要，我可为你输出：

* **Tekton Affinity Assistant 机制深入解析**
* **Pipeline YAML 示例: RWO vs RWX 最佳方案对比**
* **GKE Filestore CSI RWX PVC 配置指导**

方便你在团队培训与多租户架构设计中系统性落地与讲解。告诉我你的目标即可。


# 解决方案

新版本：

```yaml
```


老版本：
```shell
kind: ConfigMap
metadata:
  name: feature-flags
  namespace: tekton-pipelines
  labels:
    app.kubernetes.io/instance: default
    app.kubernetes.io/part-of: tekton-pipelines
data:
  # Setting this flag to "true" will prevent Tekton to create an
  # Affinity Assistant for every TaskRun sharing a PVC workspace
  #
  # The default behaviour is for Tekton to create Affinity Assistants
  #
  # See more in the workspace documentation about Affinity Assistant
  # https://github.com/tektoncd/pipeline/blob/main/docs/workspaces.md#affinity-assistant-and-specifying-workspace-order-in-a-pipeline
  # or https://github.com/tektoncd/pipeline/pull/2630 for more info.
  disable-affinity-assistant: "true"
```

直接禁用

# Reference
* [On cluster building stuck in pending state · Issue #2251 · knative/func](https://github.com/knative/func/issues/2251)
* [fix(tekton): disable affinity assistant · mindwm/mindwm-gitops@4f57e08 · GitHub](https://github.com/mindwm/mindwm-gitops/commit/4f57e0868ff1a395525fe6d041af818214ebeb84)