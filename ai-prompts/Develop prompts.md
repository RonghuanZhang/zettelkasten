```mermaid
flowchart LR
  Client 
    -->|HTTPS /create| LB(Global) 
    -->|path /create| auth-proxy-neg 
    -->|Serverless NEG| AuthProxy (Cloud Run)
    -->|validate headers against MySQL| CloudSQL (via Auth Proxy)
    -->|on success: forward /create| login-neg 
    -->|Serverless NEG| LoginService (Cloud Run)
    -->|response| Client
```
# Opensource solution prompt

```markdown
请作为资深技术选型顾问和架构师，帮助我评估业务场景下的开源解决方案。以下是需求背景：

# 业务目标
在Google Cloud Platform 上实现 Serverless 部署，包括 Service、Function 和 Job

# 关键需求
## 功能性需求
### 必须支持
1. 支持 Service 部署
	1. 支持从镜像部署
	2. 支持从 Github 部署
2. 支持 Function 部署
	1. 最起码支持 NodeJs Function
3. 支持暴露 HTTP 端点用于外部调用，或者支持 Load Balancer 内部转发

### 最好需要具备
 1. 暴露 HTTP 端点的认证鉴权
 2. 支持多租户资源隔离

## 非功能性需求
暂无

# 技术上下文
最起码支持 Google 云

# 期望输出
1. 按优先级推荐 5 个左右候选方案
2. 每个方案必须包含
	1. 核心优势匹配度分析
	2. 社区活跃度指标（GitHub stars/commit 等）
	3. 风险预警（如项目维护状态、已知缺陷等）
```
