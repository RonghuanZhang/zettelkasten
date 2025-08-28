---
"type:": fleet-note
"title:": 20250822102251-Knative controller and webhook
"id:": 20250822102301
"created:": 2025-08-22T10:23:01
url:
tags:
  - fleet-note
  - cloud-native/serverless/knative
"processed:": false
"archived:": false
---
太好，继续学习计划。下面我们把 **Controller** 和 **Webhook** 吃透；每段后给你一条“现场可验证”的动作。最后我只问你一个小问题。

# Controller（Serving 控制器）

**定位**：把你对 `ksvc` 的“期望状态”落成底层 K8s 资源与 Knative 子资源。  
**主要编排链路**：

- `Service (ksvc)` → **Configuration + Route**
    
- `Configuration` → **Revision**（当 `spec.template` 变更时生成新 Revision）
    
- `Revision` → **Deployment + Service（-private）**（包含 `queue-proxy`）
    
- `Route` → **KIngress**（再由 net-kourier-controller 下发 Envoy）
    

**它还做**：

- **镜像 digest 解析**（tag → immutable digest），保证可回滚/可复现。
    
- **流量切分/蓝绿**（按 `ksvc.spec.traffic` 权重把流量指向不同 Revisions）。
    
- **OwnerReferences** 维护层级，保证删除/回收有序。
    

**排障线索**（看 Conditions）：

- `kubectl describe ksvc <name>`：`ConfigurationsReady` / `RoutesReady`
    
- `kubectl describe revision <rev>`：镜像拉取、探针失败、资源不足等原因会直观显示。
    
- 日志：`kubectl -n knative-serving logs deploy/controller -f`
    

**动手（1 分钟）**

```bash
# 观察一次从 ksvc → config/route → revision 的映射
kubectl -n <ns> get ksvc <svc> -oyaml | sed -n '1,120p' | grep -E 'latest|traffic'
kubectl -n <ns> get configuration,route,revision -l serving.knative.dev/service=<svc>
```

# Webhook（Admission）

**定位**：在“写入 APIServer 前”统一 **默认化 + 校验 + 版本转换**。

- **默认化**：你没填的常用字段（端口、超时等）会被补上默认值。
    
- **校验**：非法值/相互冲突的配置会被拒绝（所以很多错误直接体现在 `kubectl apply` 阶段）。
    
- **版本转换**：不同 CRD 版本之间的兼容与迁移。
    

**排障线索**：

- 创建/更新报错先看 `webhook` 日志：  
    `kubectl -n knative-serving logs deploy/webhook -f`
    
- 试试 **服务器端 dry-run** 看会不会被拦：  
    `kubectl -n <ns> apply -f your.yaml --dry-run=server`
    

**动手（30 秒）**

```bash
# 看看默认值都被填了什么
kubectl -n <ns> get ksvc <svc> -oyaml
```

# 把链路串起来（可选小观察）

```bash
# 最新就绪 Revision
REV=$(kubectl -n <ns> get ksvc <svc> -o jsonpath='{.status.latestReadyRevisionName}')
echo $REV

# 该 Revision 关联的底层资源
kubectl -n <ns> get deploy,svc | grep $REV
kubectl -n <ns> get sks $REV -o jsonpath='{.status.mode}{"\n"}'   # Serve/Proxy

# 路由端
kubectl get kingress -A | head
```

---

单个小问题（只回资源名，用逗号分隔）：  
**Controller** 从一个 `Service (ksvc)` **直接**生成的两个子资源是什么？（提示：不是 Revision）