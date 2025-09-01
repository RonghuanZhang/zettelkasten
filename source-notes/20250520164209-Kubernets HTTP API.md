---
"type:": source-note
"title:": 20250520164209-Kubernets HTTP API
"id:": 20250520164218
"created:": 2025-05-20T16:42:18
source:
  - AI
url: 
tags:
  - source-note
  - kubernetes/restful-api
"processed:": false
"archived:": false
---
在 Kubernetes 中，所有资源的 HTTP 接口都遵循统一的 **RESTful API 约定**。简要来说：

* **核心组（core）资源** 使用路径前缀 `/api/v1`；
* **命名组（named API groups）** 使用 `/apis/{GROUP}/{VERSION}`；
* **命名空间作用域** 在组路径后添加 `/namespaces/{NAMESPACE}`；
* **资源类型** 直接跟在版本后，如 `/pods`、`/services`、`/services.serving.knative.dev`；
* **单个资源** 在资源类型后再加 `/{NAME}`，可选的子资源再跟 `/{SUBRESOURCE}`。

Knative 的接口（如 `GET /apis/serving.knative.dev/v1/namespaces/{ns}/services`）正是基于上述通用规范的一个定制 CRD 组。

---

## RESTful 路径结构

### 1. 核心组 vs. 命名组

* **核心（legacy）组**
  路径以 `/api/v1` 开头，用于内建的核心资源（如 Pods、Namespaces、ConfigMaps 等）([Kubernetes][1])。
* **命名组**
  自定义或扩展资源均挂在 `/apis/{GROUP}/{VERSION}`，如 `batch/v1`、`apps/v1`、或 Knative 的 `serving.knative.dev/v1`([Kubernetes][1])。

### 2. 命名空间作用域 vs. 集群作用域

* **集群作用域资源**
  路径中无 `namespaces` 段，示例：

  ```
  GET /apis/{GROUP}/{VERSION}/{RESOURCETYPE}
  ```

  列出该组下所有该类型资源，也可加 `/{NAME}` 获取单个实例([Kubernetes][2])。
* **命名空间作用域资源**
  路径需包含 `/namespaces/{NAMESPACE}`：

  ```
  GET /apis/{GROUP}/{VERSION}/namespaces/{NAMESPACE}/{RESOURCETYPE}
  ```

  查看某命名空间下的资源集合，或加 `/{NAME}` 获取具体实例([Kubernetes][2])。

---

## 通用 HTTP 方法（Verbs）映射

Kubernetes API 对资源的一致 CRUD 操作映射到标准 HTTP 方法([容器与Kubernetes][3])：

| 操作        | HTTP 方法   | 路径示例                                                      |
| --------- | --------- | --------------------------------------------------------- |
| 列表/List   | GET       | `/apis/{G}/{V}/namespaces/{ns}/{rt}`                      |
| 获取/Get    | GET       | `/apis/{G}/{V}/namespaces/{ns}/{rt}/{name}`               |
| 创建/Create | POST      | `/apis/{G}/{V}/namespaces/{ns}/{rt}`（请求体为资源定义）            |
| 更新/Update | PUT       | `/apis/{G}/{V}/namespaces/{ns}/{rt}/{name}`（完整资源定义）       |
| 补丁/Patch  | PATCH     | `/apis/{G}/{V}/namespaces/{ns}/{rt}/{name}`（Patch 格式请求体）  |
| 删除/Delete | DELETE    | `/apis/{G}/{V}/namespaces/{ns}/{rt}/{name}`               |
| 子资源       | GET/POST… | `/apis/{G}/{V}/namespaces/{ns}/{rt}/{name}/{subresource}` |

---

## Knative 示例

Knative Serving 将其 Service 定义为 CRD，注册在 `serving.knative.dev` 组：

```http
GET https://<KUBE_API_ENDPOINT>/apis/serving.knative.dev/v1/namespaces/<NAMESPACE>/services
```

* **`serving.knative.dev`**：CRD 分组名称
* **`v1`**：API 版本，对应 `apiVersion: serving.knative.dev/v1`
* **`namespaces/<NAMESPACE>/services`**：命名空间作用域的 Service 列表资源类型([Kubernetes][2])。

这与内建资源（如 `/apis/apps/v1/namespaces/default/deployments`）的访问方式完全一致。

---

## 元路径与发现机制

* **根 API 路径**

  * `/api`：列出核心组支持的版本（如 v1）
  * `/apis`：列出所有命名组（包括自定义组）([容器与Kubernetes][3])。
* **版本与资源发现**

  * `/api/v1`：返回核心资源列表
  * `/apis/{GROUP}`：返回该组支持的版本
  * `/apis/{GROUP}/{VERSION}`：返回该版本下所有资源类型及可用子资源([容器与Kubernetes][3])。

通过上述路径，客户端（如 `kubectl` 或自定义 HTTP 客户端）可以动态发现集群所支持的所有资源和版本。

---

## 核心结论

1. **HTTP 结构统一**：Knative、Core、第三方 CRD 均遵循 `/api` 或 `/apis` + 组 + 版本 + （可选的命名空间）+ 资源 + （可选名称/子资源）([Kubernetes][2], [Kubernetes][1])。
2. **自定义组即命名组**：Knative CRD 注册为 `serving.knative.dev`，访问路径与内建 Apps、Batch 等完全一致([James DeFabia][4])。
3. **资源发现**：通过访问根路径 `/api` 和 `/apis`，可枚举所有可用组、版本、资源类型，为客户端对接提供自动化支持([容器与Kubernetes][3])。

因此，你所使用的 Knative HTTP 接口，本质上 **完全通用** 于整个 Kubernetes 生态，遵循同一套 API 约定。

[1]: https://kubernetes.io/docs/reference/using-api/?utm_source=chatgpt.com "API Overview | Kubernetes"
[2]: https://kubernetes.io/docs/reference/using-api/api-concepts/?utm_source=chatgpt.com "Kubernetes API Concepts"
[3]: https://iximiuz.com/en/posts/kubernetes-api-structure-and-terminology/?utm_source=chatgpt.com "Kubernetes API Basics - Resources, Kinds, and Objects"
[4]: https://jamesdefabia.github.io/docs/api/?utm_source=chatgpt.com "Kubernetes API Overview"


# Reference
* [Kubernetes API Concepts \| Kubernetes](https://kubernetes.io/docs/reference/using-api/api-concepts/#resource-uris)