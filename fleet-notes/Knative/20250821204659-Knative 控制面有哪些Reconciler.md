---
"type:": fleet-note
"title:": 20250821204659-Knative 控制面有哪些Reconciler
"id:": 20250821204724
"created:": 2025-08-21T20:47:24
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
好问题！下面把三个 Deployment 里各自到底“管”哪些 Reconciler，和它们在控制面/数据面的定位，一口气梳理清楚。

# knative-serving/controller 里有哪些 Reconciler（控制面）

运行在 `deploy/controller` 中的进程同时装配了多套 Serving 核心 Reconciler，主要包括（按资源维度）：

* **Service**：负责把 `ksvc` 展开成 `Configuration`、`Route` 等子资源。([Go Packages][1])
* **Configuration**：根据模板生成（并滚动）**Revision**。([Go Packages][1])
* **Revision**：对接 K8s Deployment/Pod 等工作负载对象。([Go Packages][1])
* **Route**：监听 `Route`，生成/对齐 **KIngress**（Knative 自己的 Ingress 抽象），用于后续网络层实现。([Go Packages][1])
* **DomainMapping**：自定义域名到 Route 的映射。([Go Packages][1])
* **Certificate / NSCert**：与证书相关（自动 TLS 体系的一部分）。([Go Packages][2])
* **GC（Garbage Collector）**：按策略清理不活跃的 Revision。([Go Packages][1], [knative.dev][3])
* **Labeler**：在 Configuration/Route 上补齐所需标签以支撑路由/滚动逻辑等。([Go Packages][1])

> 这些 Reconciler 都在 `knative.dev/serving/pkg/reconciler/...` 下可以找到，对应的包描述里明确了职责。([Go Packages][1])

# net-kourier-controller 里有哪些（控制面 → 驱动数据面的网关）

Kourier 是 Knative 的网络层实现之一：

* **Ingress（KIngress）Reconciler**：监听 **KIngress**，把期望路由/分流等配置翻译为 Envoy 的 xDS，并下发给 **kourier-gateway（Envoy）**。Repo 的 CI 日志和源码结构里都能看到 `pkg/reconciler/ingress`。([prow.knative.dev][4], [GitHub][5])
* 同时会**监听 Endpoints/Service 变化**以更新后端目标（Revision 的 public/private SVC/Endpoints），从而在 Pod 缩扩/切换 Proxy/Serve 模式时快速反映到网关。官方文档也说明“网络层的 controller 负责 watch KIngress 并据此配置 Ingress Gateway”。([GitHub][5], [knative.dev][6])

> 这里的 **net-kourier-controller 属于控制面**；真正转发请求的是 **kourier-gateway（Envoy）**，它属于**数据面**。([GitHub][5])

# autoscaler 里有哪些（控制面）

`deploy/autoscaler`（以及可选的 `autoscaler-hpa`）装配了与伸缩相关的 Reconciler/控制逻辑：

* **KPA（PodAutoscaler）Controller**：默认的伸缩器；依据并发/ RPS 等指标在稳定/恐慌窗口内计算期望副本数，并**决定 SKS 的模式（Proxy/Serve）**。([knative.dev][7])
* **SKS（ServerlessService）Controller**：根据 KPA 的决定，调谐 **public/private Service 的 Endpoints**，从而把流量指到 **Activator**（Proxy）或直接指向 **Pods**（Serve）。该控制逻辑在 autoscaling 基类中也能看到对 **Metric** 与 **SKS** 的调谐入口（`ReconcileMetric`/`ReconcileSKS`）。([Go Packages][8])
* **Metric Controller**：为每个 PA/Revision 管理 `autoscaling.internal.knative.dev/Metric` CR，指定 `scrapeTarget`（通常是 queue-proxy 暴露的 Svc），供 autoscaler 周期抓取/聚合指标。([knative.dev][9])
* **（可选）HPA Controller**：当使用 HPA 类（`autoscaling.knative.dev/class: hpa.autoscaling.knative.dev`）时生效，对接 K8s HPA 与 CPU 指标。([knative.dev][10])

> **Autoscaler 不在请求数据路径上，属于控制面**（读取指标→决策→更新副本数与 SKS/Endpoints）。数据面是 **Activator / queue-proxy / Ingress Gateway**。官方架构页把 Activator 明确为数据面、网络层 controller 负责网关配置，侧面印证 autoscaler 的控制面角色；“Activator 在数据面”一文也给出了全链路背景。([knative.dev][6])

---

如果你想在集群里快速“对号入座”，可以这样查看：

```bash
# Controller 里正在跑哪些 reconciler（看日志关键词）
kubectl -n knative-serving logs deploy/controller | egrep -i 'service|route|revision|config(uration)|domainmapping|gc|labeler|cert'

# Kourier 控制器监听的对象
kubectl -n knative-serving logs deploy/net-kourier-controller | egrep -i 'kingress|endpoint|service|xds'

# Autoscaler 的关键动作（KPA/SKS/Metric）
kubectl -n knative-serving logs deploy/autoscaler | egrep -i 'PodAutoscaler|KPA|ServerlessService|SKS|Metric|panic|stable'
```

需要的话，我也可以给你画一张“控制面—数据面—CRD—Pod/Svc/Endpoints”对应关系小图，帮助串起来每个 Reconciler 的触发与输出关系。

[1]: https://pkg.go.dev/knative.dev/serving/pkg/reconciler "reconciler package - knative.dev/serving/pkg/reconciler - Go Packages"
[2]: https://pkg.go.dev/knative.dev/serving%40v0.46.0/pkg/reconciler/certificate "certificate package - knative.dev/serving/pkg/reconciler/certificate - Go Packages"
[3]: https://knative.dev/docs/serving/revisions/revision-admin-config-options/?utm_source=chatgpt.com "Administrator configuration options"
[4]: https://prow.knative.dev/view/gs/knative-prow/logs/nightly_net-kourier_main_periodic/1899030219327541248?utm_source=chatgpt.com "nightly_net-kourier_main_periodic #1899030219327541248"
[5]: https://github.com/knative-extensions/net-kourier "GitHub - knative-extensions/net-kourier: Purpose-built Knative Ingress implementation using just Envoy with no additional CRDs"
[6]: https://knative.dev/docs/serving/architecture/ "Architecture - Knative"
[7]: https://knative.dev/docs/serving/autoscaling/kpa-specific/?utm_source=chatgpt.com "Additional autoscaling configuration for Knative Pod ..."
[8]: https://pkg.go.dev/knative.dev/serving%40v0.46.0/pkg/reconciler/autoscaling "autoscaling package - knative.dev/serving/pkg/reconciler/autoscaling - Go Packages"
[9]: https://knative.dev/docs/serving/reference/serving-api/?utm_source=chatgpt.com "Serving API"
[10]: https://knative.dev/docs/serving/autoscaling/autoscaling-metrics/?utm_source=chatgpt.com "Configuring metrics"


# Reference