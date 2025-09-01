---
"type:": fleet-note
"title:": 20250821132132-Knative Activator
"id:": 20250821132151
"created:": 2025-08-21T13:21:51
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
太好了，咱们就把 **Activator** 拆清楚 👇（读完有个小检查题）

# 你需要的心智模型

**Activator = 冷启动接待员 + 突发流量缓冲 + 并发节流 + 伸缩信号采集者。**
当某个 Revision 没有就绪副本或需要排队时，它临时接管流量；一旦 Pod 就绪且可承载流量，会把请求“让”回给真实副本。

# 它到底做了什么

1. **接管入口（临时在链路里）**

* 当 Revision 副本=0、未就绪、或需要抑制并发时，请求会先到 Activator。
* 就绪后，通常从数据面“旁路”（不再经过 Activator），除非你开启了“突发缓冲”策略（见调优）。

2. **排队与并发控制**

* Activator 维护每个 Revision 的并发配额（结合 `containerConcurrency` 和目标并发）。
* 当后端 Pod 还没就绪或已满，会在 Activator 处**短暂排队**，防止把不可承载的流量直接打到冷容器上。

3. **伸缩信号**

* Activator 会把请求并发/速率指标上报给 Autoscaler（与各 Pod 内的 queue-proxy 指标一起），Autoscaler 决策扩/缩容，写回期望副本数。

4. **平滑切换**

* 当至少有一部分后端就绪且可接收请求，Activator 把流量切回后端；达到稳定后，它通常退出链路。

# 何时会“在路上”

* **冷启动**（scale-to-zero 后第一次请求）
* **Pod 未就绪/不可用**（探针失败、拉镜像慢）
* **突发流量**需要缓冲（你开启了“目标突发容量”后，Activator 会常驻在链路前面做缓冲）

# 常用调优点（按效果理解）

* **`containerConcurrency`（Revision 级）**：限制单 Pod 并发上限（0=不限制）。越小越稳，但需要更多副本。
* **目标并发 / 目标 RPS**：Autoscaler 的扩容目标（决定多少副本能“够用”）。
* **目标突发容量（Target Burst Capacity, TBC）**：

  * 设为 **-1** → **就绪后直接旁路 Activator**，降低额外跳数；
  * 设为 **0 或正数** → **让 Activator 常驻**，吸收瞬时洪峰（以轻微时延换更稳）。
    （可以全局配，也可按 Revision 覆盖）
* **空闲降零与保活时窗**：控制多久无流量后降到 0，以及从 0 拉起时的保守/激进程度（影响冷启动频次与时延）。

# 典型排障线索

* **首次请求很慢 / 502**：看 Activator 日志与该 Revision 的 `Ready` 条件；多数是镜像启动慢或探针过严。
* **扩容不跟手**：确认目标并发/RPS是否偏高、资源配额是否卡住、Autoscaler 指标是否可见。
* **时有 503/超时**：突发洪峰超过排队容量或后端始终未就绪；考虑开启/增大 TBC、放宽并发限制或加资源。

# 动手观察（最小实验）

1. 部署一个简单 `ksvc`（默认可降零）。
2. `kubectl logs deploy/activator -n knative-serving -f` 开一个窗口看日志。
3. **第一次请求**打到服务：你会看到 Activator 日志有该 Revision 的记录，随后集群拉起 Pod。
4. 等服务稳定后再打请求：通常不会再经过 Activator（若你启用突发缓冲，则仍会经过）。

---

**小检查（只需一句话）：**
在什么两种典型情况下，Activator 一定会进入请求路径？（你先说一种就行，我们再一起补全）
