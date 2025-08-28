---
"type:": fleet-note
"title:": 20250821133126-Knative core components
"id:": 20250821133134
"created:": 2025-08-21T13:31:34
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
很棒的目标！先给你一个**Knative Serving 核心组件速览图**，再带你按图学习。读完后我会让你做个很小的观察练习。

# 总体心智模型（从请求到容器）

用户请求 → Ingress/Kourier → **Activator**（冷启动/排队）→ **Autoscaler**（决策副本数）→ **Controller**（对 CRD 编排生效）→ **Webhook**（校验/默认值）→ 最终生成 Deployment/Service/ConfigMap 等，流量转到你的 Revision Pod。

# 组件逐个看

## 1) activator

* **核心作用**：当某个 Revision **没有活跃副本**或需要**并发抑制/排队**时接管流量；在冷启动期间缓冲请求并与 Autoscaler 协作触发扩容，待 Pod Ready 后把流量切回数据面。
* **关键点**：

  * 只在需要时入流量路径（有副本时通常旁路）。
  * 维护并发指标（请求数、RPS）供 Autoscaler 判定。
* **排障线索**：冷启动慢/502？看 `activator` 日志与目标 Revision 的 readiness；确认容器探针与镜像启动时间。

## 2) autoscaler

* **核心作用**：基于并发或 RPS 指标做**水平扩缩容决策**（到 0 也可），把期望副本数写回到对应 Revision 的 Deployment。
* **关键点**：

  * 两种常见指标：**concurrency**（默认，目标并发如 100）与 **RPS**。
  * **scale-to-zero**：长时间无流量降到 0；新请求来时由 Activator 触发唤醒。
* **排障线索**：扩容不生效？看 KPA/HPA 配置、目标并发、metrics 是否送达；确认是否被 HPA/资源上限“顶住”。

## 3) controller

* **核心作用**：Knative 的**控制平面“大管家”**。监听/对齐以下 CRD 的期望状态到 K8s 资源：

  * **Service → Configuration + Route**
  * **Configuration → Revision**
  * **Route → Ingress（Kourier/其它网关实现）**
* **关键点**：

  * 你改 `ksvc`（流量分配、环境变量、容器镜像等），Controller 负责生成/更新底层 Deployment/Service/Ingress。
* **排障线索**：状态卡在 Unknown/Not Ready？看 `controller` 日志与 `kubectl describe ksvc/route/revision` 的条件（Conditions）。

## 4) net-kourier-controller

* **核心作用**：Knative 网络层的 Kourier 实现控制器，负责把 **Route/Ingress** 规则**下沉到 Envoy**（数据面）。
* **关键点**：

  * 只管**流量路由**（域名、路径、权重）、可观测（访问日志/指标）、连接管理。
  * 与 `kourier-gateway`（Envoy Daemon/Deployment）配合。
* **排障线索**：流量不通/权重不生效？看 `net-kourier-controller` 日志与 Ingress/VirtualService（实现细节）是否同步到 Envoy。

## 5) webhook

* **核心作用**：Admission Webhook，**校验与默认化** Knative 资源（比如给 `ksvc` 填默认值、拒绝不合法配置），并提供 CRD 的 **schema 转换/版本管理**。
* **关键点**：

  * 任意创建/更新 Knative CRD 时都会经过它。
  * 失败时整个变更会被回滚。
* **排障线索**：创建 `ksvc` 报错？先看 `webhook` 日志；多半是字段非法、镜像/端口/探针配置不符规范。

# 组件交互（极简流程）

1. 你创建/更新 `Service`（ksvc）→ **webhook** 校验+默认化。
2. **controller** 生成/对齐 `Configuration/Revision/Route`，以及底层 Deployment/Service/Ingress。
3. 外部请求进来 → **Kourier/Envoy** 路由到目标 Revision。若副本=0或需要排队 → 先到 **activator**。
4. **activator** 统计负载 → **autoscaler** 计算副本数 → K8s 扩容 → 流量回到 Pod。

# 学习路径与动手点

* **看对象**：`kubectl get ksvc,route,configuration,revision -n <ns>`
* **看条件**：`kubectl describe ksvc <name>`（注意 Conditions 的 `RoutesReady/ConfigurationsReady`）
* **看日志**：`kubectl logs deploy/{activator|autoscaler|controller|net-kourier-controller|webhook} -n knative-serving`
* **看指标**：关注并发/请求率、冷启动时延、伸缩时延。
* **改一个参数试验**：把某个 Revision 的 `containerConcurrency` 或 `scale-to-zero` 时间调小，观察 autoscaler 动作与 activator 是否入链路。

---

想先从哪个开始深入？**Activator**和**Autoscaler**最直观，我们可以用一次小压测看它们如何配合伸缩。你更想先拆哪一个？（选一个就好：*Activator* 或 *Autoscaler*）
