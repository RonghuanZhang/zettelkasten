---
"type:": fleet-note
"title:": 20250822104045-Knative Service Create Flow
"id:": 20250822104052
"created:": 2025-08-22T10:40:52
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
很好，这题就是把\*\*“kn/ kubectl 创建 Knative Service”**这条链路从入口到落地讲清楚。给你一条**端到端时序\*\*（谁看谁、谁改谁），再附验证命令。

# 从 `kn service create` 到可对外服务：控制面 × 数据面 × CRD

1. **CLI → APIServer：提交 `Service.serving.knative.dev`**
   `kn`/`kubectl` 把 *Knative Service* CRD 提交到 K8s APIServer。该对象是 Knative Serving 的一等资源，用来“托管”配置、修订与路由全生命周期。([knative.dev][1])

2. **Knative Webhook（Admission）拦截**
   Knative 的 **webhook** 对该创建请求做 **默认化/校验/（必要时）版本转换**；非法配置会被拒绝，缺省项会被补上。可通过命名空间标签选择性绕过。([knative.dev][2])

3. **对象入库（etcd）**
   通过 Admission 后，APIServer 持久化该 `Service` 对象；随后一系列控制器开始“调和”。

4. **Serving Controller：把 Service 拆成 Configuration + Route**
   **controller** 监视 `Service`，创建/对齐 **`Configuration` 和 `Route`**；之后它会把 **Configuration → Revision**，并把 **Revision → Deployment（+ K8s Service 等）**，还会为伸缩创建 **KPA（PodAutoscaler）**。([knative.dev][2])

5. **镜像“Tag → Digest”解析（保障可复现）**
   在创建 **Revision** 时，controller 会把镜像 **tag 解析为不可变的 digest**，确保修订的可重复性与回滚一致性。([knative.dev][3], [Red Hat Docs][4], [Google Cloud][5])

6. **Autoscaler 侧控制器（KPA / Metric / SKS）联动**

* **KPA**（PodAutoscaler）负责按并发/RPS等指标计算期望副本，管理 “稳定/恐慌窗口” 等策略；
* **Metric** 控制器根据 KPA 生成的 **Metric** 对象配置，从 `scrapeTarget` 抓取 queue-proxy/activator 指标并按窗口聚合；
* **SKS（ServerlessService）** 控制器为每个 Revision 维护 **public / private** 两个 K8s Service，并 **切换 public 的 Endpoints**：

  * `Serve`：public Endpoints = private 的 Pod IP 列表（直达 Pod）；
  * `Proxy`：public Endpoints = Activator IP（经 Activator）。
    这些共同决定副本数与**是否经过 Activator**。([knative.dev][6], [Stack Overflow][7], [Go Packages][8])

7. **Kubernetes 内置控制器接棒**
   Deployment → ReplicaSet → **Pods** 拉起；**Endpoints 控制器**根据 Pod 就绪为 **`<rev>-private`** 写入 Pod IP；SKS 再把 public 的 Endpoints 指到 Activator 或复制 private 的 IP 列表。([Stack Overflow][7], [Go Packages][8])

8. **Route → KIngress（网络层期望）**
   **Route 控制器**根据 `ksvc.spec.traffic` 生成/更新 **`KIngress`**（域名/路径/权重/TLS 等），这是 Knative 的 Ingress 抽象，**并不直接动 Envoy**。([knative.dev][2])

9. **net-kourier-controller → Envoy（数据面生效）**
   **net-kourier-controller** 监听 **KIngress**，把路由/权重下发到 **Envoy（kourier-gateway）**。此后 Envoy 把请求按权重打到各 Revision 的 **public Service**；是否经 Activator 取决于 **SKS 模式**。([Red Hat Developer][9], [GitHub][10])

> 一句话心智图：
> `Service` →（Webhook校验/默认）→ **Controller**：`Configuration` + `Route` → `Revision` → `Deployment/KPA/...`
> `Route` → `KIngress` →（**net-kourier-controller**）→ **Envoy** → `<rev>`(public) →（**SKS** 决定）→ Activator 或 `<rev>-private`(Pod IP)。([knative.dev][2], [Red Hat Developer][9])

---

## 你可以这样“看见”每一步

* **看默认化/校验是否经过 Webhook**：
  `kubectl -n <ns> apply -f svc.yaml --dry-run=server`（若被拒，查看 `deploy/webhook` 日志）。([knative.dev][11])
* **看 Service 被拆成什么**：
  `kubectl -n <ns> get configuration,route -l serving.knative.dev/service=<svc>`（应对应你的 ksvc）。([knative.dev][1])
* **看 Revision & 镜像 digest**：
  `kubectl -n <ns> get revision <rev> -oyaml | grep image:`（应为 `@sha256:...`）。([knative.dev][3])
* **看伸缩与窗口**：
  `kubectl -n <ns> get kpa <rev> -oyaml | grep -E "metric|target"`；`kubectl -n knative-serving get cm config-autoscaler -o yaml`。([knative.dev][6])
* **看 SKS 决定是否经 Activator**：
  `kubectl -n <ns> get sks <rev> -o jsonpath='{.status.mode}{"\n"}'`；对比 `<rev>` 与 `<rev>-private` 的 Endpoints。([Stack Overflow][7])
* **看路由是否下发到 Envoy**：
  `kubectl get kingress -A`；`kubectl -n knative-serving logs deploy/net-kourier-controller -f`。([Red Hat Developer][9])

如果你愿意，我可以把这条链路画成一张小图，方便给同事培训用。

[1]: https://knative.dev/docs/serving/?utm_source=chatgpt.com "Knative Serving"
[2]: https://knative.dev/docs/serving/knative-kubernetes-services/?utm_source=chatgpt.com "Kubernetes services"
[3]: https://knative.dev/docs/serving/tag-resolution/?utm_source=chatgpt.com "Tag resolution"
[4]: https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.31/html-single/serving/index?utm_source=chatgpt.com "Serving | Red Hat OpenShift Serverless | 1.31"
[5]: https://cloud.google.com/kubernetes-engine/enterprise/knative-serving/docs/deploying?utm_source=chatgpt.com "Deploying container images | Knative serving"
[6]: https://knative.dev/docs/serving/autoscaling/kpa-specific/?utm_source=chatgpt.com "Additional autoscaling configuration for Knative Pod ..."
[7]: https://stackoverflow.com/questions/58860118/how-does-knative-servings-activator-intercept-requests-to-scaled-down-revisions?utm_source=chatgpt.com "How does Knative Serving's Activator intercept requests to ..."
[8]: https://pkg.go.dev/knative.dev/serving/pkg/apis/networking?utm_source=chatgpt.com "networking package - knative.dev/serving/pkg/apis ..."
[9]: https://developers.redhat.com/blog/2020/06/30/kourier-a-lightweight-knative-serving-ingress?utm_source=chatgpt.com "Kourier: A lightweight Knative Serving ingress"
[10]: https://github.com/knative-extensions/net-kourier?utm_source=chatgpt.com "knative-extensions/net-kourier"
[11]: https://knative.dev/docs/serving/webhook-customizations/?utm_source=chatgpt.com "Exclude namespaces from the Knative webhook"


# Reference