---
"type:": source-note
"title:": 20250519094943-Knative Routing Principle
"id:": 20250519095042
"created:": 2025-05-19T09:50:42
source:
  - AI
url: 
tags:
  - source-note
"processed:": false
"archived:": false
---
好的，既然你使用的是 **Kourier** 作为 Knative 的网络层组件（而不是 Istio），那么整个访问流程相对更加轻量、简洁，下面我会专门从 **Kourier 的角度** 重新帮你详细梳理一下访问流程。

---

## 🎯 背景假设

你创建了一个 Knative Service，名称为 `rhz-greetings`，Namespace 为 `default`，访问地址为：

```
http://rhz-greetings.default.hiverun.io
```

你使用的是 **Kourier** 作为 Ingress 实现。

---

## 🧭 请求从浏览器到 Pod 的完整路径

我们将整个路径拆成 **七大阶段**，从请求发出 → 到你的容器实际执行业务逻辑。

---

### **1. DNS 解析 & 请求发出**

- 当客户端访问 `http://rhz-greetings.default.hiverun.io`：
    
    - 浏览器会通过 DNS 将域名解析为一个 **IP 地址**。
        
    - 这个 IP 地址通常指向一个 LoadBalancer 类型的 Service，叫做：
        
        ```
        kourier-ingress
        Namespace: kourier-system
        ```
        

你可以查看它的地址：

```bash
kubectl get svc kourier --namespace kourier-system
```

---

### **2. 外部流量进入 Kourier Gateway**

- 上述地址会将请求转发到集群内的 Kourier 网关 Pod：
    
    ```
    Deployment: 3scale-kourier-gateway
    ```
    
- 它是一个基于 **Envoy** 的边缘代理，负责接收所有外部进来的 HTTP 请求。
    

---

### **3. Knative 创建 KIngress -> Kourier 路由规则**

- 当你创建一个 Knative Service 时，Knative 会自动创建一个 `Ingress` 资源（`kind: Ingress.serving.knative.dev`），我们简称它为 **KIngress**。
    
- Knative Controller 会将这个 `KIngress` 资源，转换成 Kourier 能识别的配置，最终转化为一组 **Envoy 路由规则**。
    

这些规则包括：

- 匹配 host（如 `rhz-greetings.default.hiverun.io`）
    
- 匹配 path
    
- 转发到目标 Revision 的 Pod 或 Activator（如果 scale=0）
    

---

### **4. Kourier 判断目标 Revision 是否可用**

#### 有两种情况：

#### ✅ **目标 Revision 已在运行**：

- Kourier 会将请求直接转发到目标 Revision Pod 的 **Queue-Proxy 容器**。
    

#### ❌ **目标 Revision 缩容为 0（scale to zero）**：

- Kourier 会将请求转发给 **Activator**：
    
    ```
    Pod: activator
    Namespace: knative-serving
    ```
    

---

### **5. Activator 处理（仅当 Revision 为 0）**

当请求进入 Activator 时：

- Activator 会临时缓存请求。
    
- 并向 **Autoscaler** 发送请求，触发目标 Revision **冷启动（scale from 0）**。
    
- 等 Pod 启动并就绪后，Activator 会将缓存的请求转发给新的 Pod。
    
- 后续流量不再经过 Activator，而是直接由 Kourier 发送到 Pod。
    

---

### **6. 到达目标 Pod 的 Queue-Proxy 容器**

- 所有流量（无论是 Kourier 直发的，还是 Activator 转发的）最终都会进到 Pod 的 `queue-proxy` 容器。
    
- Queue-proxy 是 Knative 自动注入的 sidecar，主要功能：
    
    - 控制并发数（配合 `containerConcurrency`）
        
    - 上报 metrics 给 autoscaler
        
    - 做健康探针和超时处理
        
    - 最后代理请求给你的应用容器（即你的 Spring Boot / Java 容器）
        

---

### **7. 应用容器处理业务请求**

- 应用容器收到请求，执行业务逻辑，处理数据库连接、接口调用、生成响应。
    
- 然后将响应结果返回 → queue-proxy → Kourier → 用户。
    

---

## 🔄 数据流图（简化版）

```text
客户端
   |
   | (1) DNS -> rhz-greetings.default.hiverun.io
   |
   v
LoadBalancer (Service: kourier)
   |
   v
3scale-kourier-gateway (Envoy)
   |
   |--> Revision running? ---- Yes ---> Pod (queue-proxy -> app container)
   |                        |
   |                        No
   |                        |
   |                     v
   |                 Activator
   |                     |
   |             cold start target Pod
   |                     |
   |              forward to new Pod
   |
   v
客户端收到响应
```

---

## ✅ 小结（基于 Kourier）

|阶段|描述|
|---|---|
|1. DNS 解析|自定义域名解析到 `kourier` 的 LB 地址|
|2. Kourier Gateway|基于 Envoy 的代理，接收请求|
|3. 路由规则|Knative 的 KIngress 生成的 Envoy 路由规则|
|4. 判定 Revision|是否已启动，如果没有则走 Activator|
|5. Activator|缓冲请求 + 触发 ScaleUp|
|6. Queue-Proxy|控制并发 & 代理请求|
|7. App 容器|执行业务逻辑并返回响应|

---

如果你需要我进一步结合实际 `kubectl` 命令或者你自己的配置（比如你用了 `DomainMapping` 或者你想搞清楚具体是哪个 Envoy 路由规则生效），我也可以继续深入。

# Reference


