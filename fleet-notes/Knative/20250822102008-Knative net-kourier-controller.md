---
"type:": fleet-note
"title:": 20250822102008-Knative net-kourier-controller
"id:": 20250822102018
"created:": 2025-08-22T10:20:18
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
太好了，来把 **net-kourier-controller** 的“职责边界 + 实操观察点”吃透。先给你心智模型，再给你 5 个现场排查/验证动作。最后我只问你一个小问题来确认掌握。

# 心智模型（谁做决定、谁来执行）

* **net-kourier-controller = 控制面**：监听 Knative 的 **Kingress**（`ingress.networking.knative.dev`，俗称 *KIngress*），把路由/证书等配置**下发到 Envoy（kourier-gateway）**，从而实现域名/路径/权重等转发。([GitHub][1], [Red Hat Developer][2])
* **kourier-gateway = 数据面（Envoy）**：真正承载入口流量，按照 controller 下发的配置把请求转发到后端。常见部署是 **`kourier-system`** 命名空间里的 Envoy 网关与 Service（如 `kourier` / `kourier-internal`）。([Go Packages][3])
* **是否经过 Activator** 不由 kourier 决定，而是 **Serving 内部（SKS 切换 Endpoints）** 决定；kourier 只按后端 Service 的 Endpoints 转发。你前面学过这点。
* **为什么选 Kourier**：轻量、无额外 CRD、只由 Envoy + 控制面组成，是 Istio 的简化替代。([GitHub][4])

# net-kourier-controller 具体做什么

1. **对接 Knative Ingress 抽象**：监听/对齐 **KIngress** 及相关网络配置（域名、TLS、权重拆分等），生成 Envoy 的 Listener/Route/Cluster 配置。([GitHub][1], [Red Hat Developer][2])
2. **推动到 Envoy**：把最新配置推送给 **kourier-gateway**，确保流量路由与权重更新实时生效。([Red Hat Developer][2])
3. **TLS 与基础观测**：支持基本 TLS 终止、指标与日志（“够用但不重”）。([Knative][5])
4. **IngressClass 绑定**：当 `ingress-class` 设为 `kourier.ingress.networking.knative.dev` 时，由 net-kourier-controller 来接管。([Knative][5], [Go Packages][3])
5. **可调项**：可通过 **`kourier-bootstrap` ConfigMap** 调整 Envoy 启动配置（例如 admin 端口、监听器、特性开关等）。([Knative][6])

# 现场怎么“看见”它在工作（5 步）

1. **确认 IngressClass 与控制器**

```bash
# 看 operator/Serving CR 是否启用 Kourier，并把 ingress-class 指到 kourier
kubectl -n knative-serving get knativeserving.operator.knative.dev knative-serving -o yaml | grep -A3 ingress:
kubectl -n knative-serving get cm config-network -o yaml | grep ingress-class
```

> 预期为 `kourier.ingress.networking.knative.dev`。([Knative][5])

2. **查 KIngress（Knative 的 Ingress 抽象）状态**

```bash
kubectl get kingress -A
kubectl -n <ns> describe kingress <name>
```

> 这是 net-kourier-controller 的输入对象；状态正常时才会把配置下发到 Envoy。([GitHub][1])

3. **看 controller 日志（是否在调和）**

```bash
kubectl -n knative-serving logs deploy/net-kourier-controller -f
```

> 这里能看到它如何根据 KIngress 生成/更新路由。([Red Hat Documentation][7])

4. **定位网关与 Service**

```bash
kubectl get ns | grep kourier
kubectl -n kourier-system get svc,deploy | grep kourier
```

> 常见是 `kourier-system` 命名空间里有 `kourier`（对外）和 `kourier-internal`（集群内）服务名，后端是 `kourier-gateway`(Envoy)。([Go Packages][3])

5. **（可选）直接看 Envoy 的实时配置**

```bash
# 端口可能随 bootstrap 而不同；Envoy admin 常见是 19000
kubectl -n kourier-system port-forward deploy/kourier-gateway 19000:19000
curl localhost:19000/config_dump | jq '.configs[] | .@type? // empty'
```

> admin 端口取决于 `kourier-bootstrap`，上面只是常见值；需要时可先查看该 ConfigMap 或 Deployment 参数。([Knative][6], [Envoy Gateway][8])

# 常见排障线索

* **域名解析不到/无外网入口**：`Service/kourier` 的外部 IP 或 CNAME 是否准备好；Operator 安装方式下可直接 `kubectl -n knative-serving get svc kourier` 查看（不同发行方式会有差异）。([Knative][5])
* **权重/路径不生效**：`KIngress` 条目是否更新到 **net-kourier-controller**（看日志）并同步到 Envoy（看 `config_dump`）。([Red Hat Developer][2])
* **HTTPS/TLS 问题**：检查证书 Secret 绑定的 `KIngress`/域名是否匹配；Kourier 支持基本 TLS 终止。([Knative][5])

---

单个小问题（只回一个词即可）：
net-kourier-controller 主要监听并据此下发 Envoy 配置的那个 Knative CRD，简称叫什么？（提示：**KIngress**）

[1]: https://github.com/knative/networking?utm_source=chatgpt.com "knative/networking"
[2]: https://developers.redhat.com/blog/2020/06/30/kourier-a-lightweight-knative-serving-ingress?utm_source=chatgpt.com "Kourier: A lightweight Knative Serving ingress"
[3]: https://pkg.go.dev/knative.dev/net-kourier/pkg/config?utm_source=chatgpt.com "config package - knative.dev/net-kourier ..."
[4]: https://github.com/knative-extensions/net-kourier?utm_source=chatgpt.com "knative-extensions/net-kourier"
[5]: https://knative.dev/docs/install/operator/knative-with-operators/?utm_source=chatgpt.com "Install by using the Knative Operator"
[6]: https://knative.dev/docs/install/operator/configuring-serving-cr/?utm_source=chatgpt.com "Configuring Knative Serving CRDs"
[7]: https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.34/html/serving/kourier-and-istio-ingresses?utm_source=chatgpt.com "Chapter 6. Kourier and Istio ingresses"
[8]: https://gateway.envoyproxy.io/v1.3/troubleshooting/envoy-proxy-admin-interface/?utm_source=chatgpt.com "Advanced: Envoy Proxy Admin Interface"
