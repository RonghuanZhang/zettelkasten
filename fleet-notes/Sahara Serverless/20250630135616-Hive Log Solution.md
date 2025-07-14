---
"type:": fleet-note
"title:": 20250630135616-Hive Log Solution
"id:": 20250630135623
"created:": 2025-06-30T13:56:23
url: 
tags:
  - fleet-note
  - kubernetes
  - fluent-bit
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---

## Vercel Research

CLI stream logs and website logs.

![image.png](https://images.hnzhrh.com/note/20250704143741111.png)

Loki + Grafana

# 安装 Knative Fluent Bit

## 安装命令

```shell
curl -LO https://raw.githubusercontent.com/knative/docs/main/docs/serving/observability/logging/fluent-bit-collector.yaml
```

## YAML 资源文件

```yaml
apiVersion: v1  
kind: Namespace  
metadata:  
  name: logging  
---  
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: log-collector-config  
  namespace: logging  
  labels:  
    k8s-app: log-collector  
data:  
  # Configuration files: server, input, filters and output  
  # ======================================================  fluent-bit.conf: |  
    [SERVICE]  
        Flush         1  
        Log_Level     info  
        Daemon        off  
        HTTP_Server   On  
        HTTP_Listen   0.0.0.0  
        HTTP_Port     2020  
  
    @INCLUDE input-forward.conf  
    @INCLUDE filter-simplify.conf  
    @INCLUDE output-files.conf  
  
  input-forward.conf: |  
    [INPUT]  
        Name              forward  
        Port              ${FLUENT_PORT}  
  
  # This filter simplifies the fluentbit "tag" which is used to choose the  
  # output filename. First match Knative services, then match pods with an "app"  # label, then use the pod name.  filter-simplify.conf: |  
    [FILTER]  
      Name      rewrite_tag  
      Match     kube.*  
      Rule      $kubernetes['labels']['serving.knative.dev/configuration'] ^(.+)$ knative.$kubernetes['namespace_name'].$1.log false  
  
    [FILTER]  
      Name      rewrite_tag  
      Match     kube.*  
      Rule      $kubernetes['labels']['app'] ^(.+)$ app.$kubernetes['namespace_name'].$1.log false  
  
    [FILTER]  
      Name      rewrite_tag  
      Match     kube.*  
      Rule      $kubernetes['pod_name'] ^(.+)$ pod.$kubernetes['namespace_name'].$1.log false  
  
  output-files.conf: |  
    [OUTPUT]  
        Name            file  
        Path            /logs  
        Format          plain  
        Match           *  
---  
apiVersion: v1  
kind: ConfigMap  
metadata:  
  name: log-nginx-config  
  namespace: logging  
  labels:  
    k8s-app: log-collector  
data:  
  nginx.conf: |  
    events {  
    }  
  
    http {  
      include mime.types;  
      default_type application/octet-stream;  
      sendfile on;  
  
      server {  
        listen 8080;  
        server_name localhost;  
  
        location / {  
          root html;  
          autoindex on;  
        }  
      }  
    }  
  mime.types: |  
    types {  
      text/html        html htm shtml;  
      text/plain       txt log;  
    }  
---  
apiVersion: v1  
kind: Service  
metadata:  
  name: log-collector  
  namespace: logging  
  labels:  
    k8s-apps: log-collector  
spec:  
  ports:  
  - port: 24224  
    name: forward  
  - port: 80  
    name: viewer  
    targetPort: viewer  
  selector:  
    k8s-apps: log-collector  
---  
apiVersion: apps/v1  
kind: StatefulSet  
metadata:  
  name: log-collector  
  namespace: logging  
  labels:  
    k8s-apps: log-collector  
spec:  
  selector:  
    matchLabels:  
      k8s-apps: log-collector  
  serviceName: log-collector  
  replicas: 1  
  template:  
    metadata:  
      labels:  
        k8s-apps: log-collector  
    spec:  
      terminationGracePeriodSeconds: 10  
      containers:  
      - name: fluent-bit  
        image: fluent/fluent-bit:1.5  
        imagePullPolicy: Always  
        ports:  
        - containerPort: 24224  
        volumeMounts:  
        - name: logs  
          mountPath: /logs  
        - name: fluentd-config  
          mountPath: /fluent-bit/etc/  
        env:  
        - name: FLUENT_PORT  
          value: "24224"  
      - name: nginx  
        image: registry.k8s.io/nginx-slim:0.8  
        ports:  
        - containerPort: 8080  
          name: viewer  
        volumeMounts:  
        - name: logs  
          mountPath: /usr/share/nginx/html  
        - name: nginx-config  
          mountPath: /etc/nginx  
      volumes:  
        - name: fluentd-config  
          configMap:  
            name: log-collector-config  
        - name: nginx-config  
          configMap:  
            name: log-nginx-config  
  volumeClaimTemplates:  
  - metadata:  
      name: logs  
    spec:  
      accessModes: [ "ReadWriteOnce" ]  
      resources:  
        requests:  
          storage: 40Gi
```

## YAML 资源文件 AI 解释

以下是对你这份 **Fluent Bit + Nginx 日志收集与展示** Kubernetes 配置文件的详细逐段分解与解释，涵盖用途、功能及配置要点，帮助你彻底理解：

---

## 📝 **1. Namespace 定义**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: logging
```

🔹 **功能**：创建一个名为 `logging` 的命名空间，用于隔离部署日志收集相关资源（ConfigMap, Service, StatefulSet）。

---

## 📝 **2. ConfigMap: Fluent Bit 配置**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-collector-config
  namespace: logging
data:
  fluent-bit.conf: | ...
  input-forward.conf: | ...
  filter-simplify.conf: | ...
  output-files.conf: | ...
```

### **作用**：

存储 Fluent Bit 的配置文件，包括：

#### ✅ **fluent-bit.conf**

主配置文件，定义服务全局配置和引入子配置。

```ini
[SERVICE]
    Flush         1               # 每 1 秒 flush 一次
    Log_Level     info            # 日志级别
    Daemon        off             # 以前台模式运行
    HTTP_Server   On              # 开启 HTTP server
    HTTP_Listen   0.0.0.0         # 监听地址
    HTTP_Port     2020            # 监听端口

@INCLUDE input-forward.conf
@INCLUDE filter-simplify.conf
@INCLUDE output-files.conf
```

---

#### ✅ **input-forward.conf**

配置 **forward 输入插件**，监听 `24224` 端口接收日志。

```ini
[INPUT]
    Name              forward
    Port              ${FLUENT_PORT}
```

> 这里使用了环境变量 `FLUENT_PORT=24224`。

---

#### ✅ **filter-simplify.conf**

配置 **标签重写 filter**，用于根据日志的 Kubernetes 元信息重写 tag，以决定最终的输出文件名。

包含三条规则：

1. 如果 pod 有 Knative configuration label，则输出文件名为 `knative.<namespace>.<configuration>.log`
2. 如果 pod 有 `app` label，则输出 `app.<namespace>.<app>.log`
3. 否则使用 pod 名称，输出 `pod.<namespace>.<pod>.log`

例如：

* knative.default.helloworld.log
* app.default.nginx.log
* pod.default.nginx-xxx.log

---

#### ✅ **output-files.conf**

配置 **输出插件 file**，将所有匹配到的日志输出为文件：

```ini
[OUTPUT]
    Name            file
    Path            /logs
    Format          plain
    Match           *
```

🔹 **输出路径**：`/logs`

---

## 📝 **3. ConfigMap: Nginx 配置**

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: log-nginx-config
  namespace: logging
data:
  nginx.conf: | ...
  mime.types: | ...
```

### **作用**：

定义 Nginx 用于 **暴露 /logs 文件夹内容** 的配置，提供浏览日志文件的 HTTP 接口。

#### ✅ **nginx.conf**

* 监听 8080 端口
* 根目录为 `/usr/share/nginx/html`（挂载在 /logs）
* 开启 `autoindex on`，可以浏览目录

---

#### ✅ **mime.types**

增加对 `.log` 文件的 MIME type 支持：

```nginx
types {
  text/html        html htm shtml;
  text/plain       txt log;
}
```

---

## 📝 **4. Service 定义**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: log-collector
spec:
  ports:
  - port: 24224
    name: forward
  - port: 80
    name: viewer
    targetPort: viewer
  selector:
    k8s-apps: log-collector
```

### **作用**：

定义一个 Service，暴露 StatefulSet 的两个端口：

1. `24224` - Fluent Bit forward 输入
2. `80` - Nginx viewer (通过 targetPort: viewer，实际 containerPort: 8080)

---

## 📝 **5. StatefulSet: log-collector**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: log-collector
spec:
  serviceName: log-collector
  replicas: 1
  selector:
    matchLabels:
      k8s-apps: log-collector
  template:
    spec:
      containers:
      - name: fluent-bit
        image: fluent/fluent-bit:1.5
        ports:
        - containerPort: 24224
        volumeMounts:
        - name: logs
          mountPath: /logs
        - name: fluentd-config
          mountPath: /fluent-bit/etc/
        env:
        - name: FLUENT_PORT
          value: "24224"

      - name: nginx
        image: registry.k8s.io/nginx-slim:0.8
        ports:
        - containerPort: 8080
          name: viewer
        volumeMounts:
        - name: logs
          mountPath: /usr/share/nginx/html
        - name: nginx-config
          mountPath: /etc/nginx
```

### **核心点**：

🔹 **副本数**：1
🔹 **包含容器**：

1. **fluent-bit**

   * 输入端口: 24224
   * 配置路径: /fluent-bit/etc/ (挂载 log-collector-config ConfigMap)
   * 日志输出到 `/logs`

2. **nginx**

   * 暴露 8080 (viewer)
   * 配置路径: /etc/nginx (挂载 log-nginx-config ConfigMap)
   * 根目录 `/usr/share/nginx/html` 对应 `/logs`（与 fluent-bit 共享）

---

### **挂载 Volume**:

* **volumeClaimTemplates** 定义 `logs` PVC (40Gi) 用于持久化日志。

```yaml
volumeClaimTemplates:
- metadata:
    name: logs
  spec:
    accessModes: [ "ReadWriteOnce" ]
    resources:
      requests:
        storage: 40Gi
```

---

## 📝 **🔑 总结**

| 组成                    | 功能                               |
| --------------------- | -------------------------------- |
| **Namespace logging** | 隔离所有日志收集资源                       |
| **Fluent Bit**        | 收集各 Pod 日志，按标签重写输出为文件            |
| **Nginx**             | 提供 HTTP 文件浏览服务                   |
| **Service**           | 暴露 Fluent Bit 和 Nginx            |
| **StatefulSet**       | 运行 fluent-bit + nginx 并挂载持久卷存储日志 |

---

### ✅ **最终效果**

* 其他 Pod 将日志通过 fluent-forward 发送到 `log-collector.logging.svc:24224`
* Fluent Bit 收集、重写 tag 并将日志写入 PVC 挂载的 `/logs` 目录
* Nginx 将该目录暴露在 8080 端口（或通过 Service 80 端口）供用户浏览和下载

---

如需我帮你：

1. **优化 ConfigMap 结构**
2. **改写为 Helm Chart**
3. **集成 ELK/Opensearch 输出**
4. **增加Ingress暴露配置**

请告诉我，我会为你的 Logging Stack 提供最佳实践与可维护方案。

# Reference
* [vercel logs](https://vercel.com/docs/cli/logs)
* [How do I store logs on Vercel?](https://vercel.com/guides/how-do-i-store-logs-on-vercel)
* [How to send Logs to Loki using Fluent Bit](https://chronosphere.io/learn/fluent-bit-loki/)
* [Title Unavailable \| Site Unreachable](https://knative-sample.com/40-cloud-native-practice/10-log-and-alert/)