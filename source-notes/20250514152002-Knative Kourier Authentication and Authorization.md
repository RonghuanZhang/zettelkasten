---
"type:": source-note
"title:": 20250514152002-Knative Kourier Authentication and Authorization
"id:": 20250514152034
"created:": 2025-05-14T15:20:34
source:
  - AI
url: 
tags:
  - source-note
"processed:": false
"archived:": false
---
以下内容概览了 Kourier 在认证与鉴权（AuthN/AuthZ）方面的能力、常见用法、配置示例及局限性，帮助您判断 Kourier 是否满足业务需求，以及在何种场景下需要借助外部组件或 Service Mesh 来增强安全性。

## 概括

Kourier 本身**不提供内置的认证或授权**逻辑，但通过集成 Envoy 的 External Authorization 过滤器（`envoy.filters.http.ext_authz`），可以将流量转发到外部授权服务，从而实现自定义的鉴权流程，比如 JWT 校验或 OIDC 授权 ([GitHub](https://github.com/knative-extensions/net-kourier "GitHub - knative-extensions/net-kourier: Purpose-built Knative Ingress implementation using just Envoy with no additional CRDs"), [GitHub](https://github.com/knative-extensions/net-kourier/issues/1189?utm_source=chatgpt.com "Provide more authentication mechanisms? · Issue #1189 - GitHub"))。此外，Kourier 支持基础的 TLS 终端，但**不包含内置的客户端证书验证或 JWT 验证功能** ([docs.openshift.com](https://docs.openshift.com/serverless/1.33/knative-serving/kourier-and-istio-ingresses.html "Chapter 5. Kourier and Istio ingresses | Red Hat Product Documentation"))。若需要更全面的认证能力（如自动处理 JWT、策略管理、mTLS 等），则通常需将 Kourier 与 Service Mesh（如 Istio/OpenShift Service Mesh）或外部授权系统联合使用 ([docs.openshift.com](https://docs.openshift.com/serverless/1.31/knative-serving/config-access/serverless-ossm-with-kourier-jwt.html?utm_source=chatgpt.com "Chapter 6. Configuring access to Knative services"), [红帽文档](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.31/html/serving/configuring-access-to-knative-services?utm_source=chatgpt.com "Chapter 6. Configuring access to Knative services"))。

## 支持的认证与鉴权方式

### 1. External Authorization（外部授权）

Kourier 利用 Envoy 的 `ext_authz` 过滤器，将请求拦截并转发到外部授权服务。您可以在 `net-kourier-controller` 部署中通过环境变量启用并配置：

```yaml
env:
  - name: KOURIER_EXTAUTHZ_HOST     # 授权服务地址，如 my-authz.default.svc.cluster.local:5000
  - name: KOURIER_EXTAUTHZ_PROTOCOL # grpc、http 或 https，默认 grpc
  - name: KOURIER_EXTAUTHZ_TIMEOUT  # 等待外部服务响应的最大毫秒数，默认 2000
  - name: KOURIER_EXTAUTHZ_FAILUREMODEALLOW # auth 服务不可用时是否放行(true/false)
  # 其他可选参数：PATHPREFIX、MAXREQUESTBYTES、PACKASBYTES 等
```

这样，当流量到达 Kourier 时，Envoy 会先调用外部授权服务进行决策，只有获得允许的请求才会被路由至后端 ([GitHub](https://github.com/knative-extensions/net-kourier "GitHub - knative-extensions/net-kourier: Purpose-built Knative Ingress implementation using just Envoy with no additional CRDs"), [Go Packages](https://pkg.go.dev/knative.dev/net-kourier/pkg/config "config package - knative.dev/net-kourier/pkg/config - Go Packages"))。

- **典型用途**：调用自定义微服务来校验 JWT、API Key、OIDC Token，或查询权限中心。
    

### 2. 与 Service Mesh 集成的 JWT 鉴权

在 OpenShift Serverless 或 Kubernetes 集群中，若已部署 Istio/OpenShift Service Mesh，可让 Service Mesh 处理细粒度的身份验证与授权：

1. 为目标 Pod 启用 Sidecar 注入（`sidecar.istio.io/inject: "true"`）。
    
2. 创建 `RequestAuthentication`（JWT 校验）和 `AuthorizationPolicy`（访问控制）等 CRD。
    
3. 让 Kourier 仅负责流量转发，Service Mesh 在入口（或应用侧）完成 Token 校验与策略执行 ([红帽文档](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.31/html/serving/configuring-access-to-knative-services?utm_source=chatgpt.com "Chapter 6. Configuring access to Knative services"), [docs.openshift.com](https://docs.openshift.com/serverless/1.33/knative-serving/kourier-and-istio-ingresses.html "Chapter 5. Kourier and Istio ingresses | Red Hat Product Documentation"))。
    

> **注意**：若在系统命名空间（如 `knative-serving` 或 `knative-serving-ingress`）启用 Sidecar，Kourier 不支持对这些 Pod 的 Sidecar 注入；需参考 OpenShift 官方集成文档 ([docs.openshift.com](https://docs.openshift.com/serverless/1.31/knative-serving/config-access/serverless-ossm-with-kourier-jwt.html?utm_source=chatgpt.com "Chapter 6. Configuring access to Knative services"))。

## 配置示例

### 示例：启用 External Authorization

```bash
kubectl -n knative-serving patch deployment net-kourier-controller \
  --type merge \
  -p '{
    "spec":{"template":{"spec":{"containers":[{"name":"kourier","env":[
      {"name":"KOURIER_EXTAUTHZ_HOST","value":"authz.default.svc.cluster.local:5000"},
      {"name":"KOURIER_EXTAUTHZ_PROTOCOL","value":"grpc"},
      {"name":"KOURIER_EXTAUTHZ_TIMEOUT","value":"3000"},
      {"name":"KOURIER_EXTAUTHZ_FAILUREMODEALLOW","value":"false"}
    ]}]}]}}}'
```

上述配置将在每次请求到达 Kourier Gateway 时，调用 `authz.default.svc.cluster.local:5000` 进行鉴权 ([GitHub](https://github.com/knative-extensions/net-kourier "GitHub - knative-extensions/net-kourier: Purpose-built Knative Ingress implementation using just Envoy with no additional CRDs"))。

### 示例：与 Istio 集成做 JWT 鉴权

```yaml
apiVersion: serving.knative.dev/v1
kind: Service
metadata:
  name: my-service
  annotations:
    sidecar.istio.io/inject: "true"
spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/rewriteAppHTTPProbers: "true"
    spec:
      containers:
      - image: ghcr.io/my-org/my-app
```

然后在同命名空间添加：

```yaml
apiVersion: security.istio.io/v1beta1
kind: RequestAuthentication
metadata:
  name: jwt-auth
spec:
  selector:
    matchLabels:
      serving.knative.dev/service: my-service
  jwtRules:
  - issuer: "https://issuer.example.com/"
    jwksUri: "https://issuer.example.com/.well-known/jwks.json"
---
apiVersion: security.istio.io/v1beta1
kind: AuthorizationPolicy
metadata:
  name: allow-jwt
spec:
  selector:
    matchLabels:
      serving.knative.dev/service: my-service
  rules:
  - from:
    - source:
        requestPrincipals: ["*"]
```

如此即可让 Istio 在入口即对 JWT 进行验证与断言 ([红帽文档](https://docs.redhat.com/en/documentation/red_hat_openshift_serverless/1.31/html/serving/configuring-access-to-knative-services?utm_source=chatgpt.com "Chapter 6. Configuring access to Knative services"))。

## 局限性与选型建议

- **无内置 AuthN/AuthZ**：Kourier 仅做流量转发与基础 TLS 终端 ([docs.openshift.com](https://docs.openshift.com/serverless/1.33/knative-serving/kourier-and-istio-ingresses.html "Chapter 5. Kourier and Istio ingresses | Red Hat Product Documentation"))。如需内置 Token 验证、策略管理、审计日志等，请使用 Istio/OpenShift Service Mesh 或 API Gateway。
    
- **依赖外部组件**：External Authorization 需额外开发或部署鉴权服务（如自研微服务、Kong、OAuth2 Proxy 等）。
    
- **性能与复杂度**：外部鉴权会增加额外网络调用，配置失败或服务不可用可能影响请求延迟与可用性。可通过配置 `FAILUREMODEALLOW` 和合理的 `TIMEOUT` 降低风险。
    
- **未来支持**：社区已有讨论在 Kourier 中原生支持 JWT/OIDC 等 ([GitHub](https://github.com/knative-extensions/net-kourier/issues/1189?utm_source=chatgpt.com "Provide more authentication mechanisms? · Issue #1189 - GitHub"))，可关注 net-kourier 项目更新。
    

---

综上，**Kourier 本身不提供内置的认证鉴权功能**，但通过 Envoy 的 External Authorization 机制可灵活接入自定义或第三方鉴权服务；若对认证、授权、安全策略有更高要求，推荐与 Service Mesh（例如 Istio/OpenShift Service Mesh）配合使用，以获得端到端的安全保障。

# Reference