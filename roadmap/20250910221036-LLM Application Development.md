---
"type:": sop
"title:": 20250910221036-LLM Application Development
"id:": 20250910221044
"created:": 2025-09-10T22:10:44
url:
tags:
  - llm
  - roadmap
---

太好了，Java 背景+想加上 Go 做 AI 应用，这组合很能打。下面给你一份「全景图 + 由浅入深路线」，都围绕主流技术栈，而且尽量贴合你现有的 K8s/Knative/DDD 经验。

---

# 全景模块（一图看全）

**目标**：做出可上线、可观测、可扩展的企业级 AI 应用（RAG/Agent/多租户）。

* **A. 模型与推理**

  * 云端闭源：OpenAI / Azure OpenAI、Anthropic、Google
  * 开源/自托管：vLLM、SGLang、Ollama、LocalAI（OpenAI-compatible）
  * 模型服务：**KServe**（Knative 友好）、BentoML、Ray Serve
* **B. 应用编排（Java/Go）**

  * Java：**Spring Boot 3**、**Spring AI**、**LangChain4j**、Micrometer、gRPC/WebFlux
  * Go：**sashabaranov/go-openai**、**langchaingo**、chi/fiber、grpc-go、fx/wire（DI）
* **C. RAG 与数据层**

  * 向量库：**Milvus / Qdrant / Weaviate / pgvector**
  * 检索：BM25（Elasticsearch/OpenSearch）、Hybrid（稀疏+稠密）
  * 解析/清洗：Apache Tika（Java）、unstructured（HTTP 集成）
* **D. 代理/工具调用（Agents）**

  * 工具函数调用 / JSON 模式约束
  * 工作流：Temporal / Argo Workflows（可选）
* **E. 工程与平台**

  * **Kubernetes + Knative**、**KServe**
  * CI/CD：**Tekton** + Buildpacks；镜像：Harbor
  * 观测：**OpenTelemetry**、Prometheus、Grafana、Loki/Tempo/Jaeger
  * 配置与密钥：K8s Secret / Vault
  * 多租户：命名空间/网关隔离、配额、账务与成本跟踪
* **F. 评测与安全**

  * 质量评测：对话/RAG 准确率（检索命中率、答案忠实度）
  * 安全：敏感信息抽取与脱敏、越狱防护、权限与审计

---

# 由浅入深学习路线（阶段式）

## 0️⃣ 预热（打底与工具）

* LLM 基础：token、上下文窗口、温度/采样、系统/工具调用
* 最小闭环：用 Java/Go 直连一个 OpenAI-compatible API，完成**流式对话**与**函数调用**
  练习：实现“根据城市返回天气”的**函数调用**示例（伪造工具返回）。

> Checkpoint：你能说清“API直连”和“自托管推理服务”的差异与取舍。

---

## 1️⃣ 入门应用（Chat/RAG 雏形）

* **Java 路线**：Spring Boot + **Spring AI** 或 **LangChain4j**

  * PromptTemplate、ChatClient/Chain、消息记忆、调用日志
  * 简易 RAG：文件上传 → 拆分 → 向量化 → 向量库检索 → 组装上下文
* **Go 路线**：chi/fiber + **go-openai** 或 **langchaingo**

  * SSE 流式输出、工具调用（function/tool）、简单 RAG

**小项目 A**：企业知识库问答（单租户）

* 选一个向量库（Qdrant/pgvector），支持：上传 PDF → 检索 → 答案可追溯（source 引用）
* 指标：检索条数、延迟、token 成本、请求成功率

---

## 2️⃣ 进阶工程（可观测/可扩展）

* **观测**：OpenTelemetry Trace + Micrometer/Prometheus 指标（请求数、延迟、失败率、token 使用量）
* **可靠性**：重试/超时/熔断；提示词版本化与“黄金样本”回归测试
* **RAG 强化**：Hybrid 检索、重排序（可由 Python 服务提供 HTTP 重排）、Chunk 策略与去重
* **代理/工具**：对接内部 REST/gRPC、DB 查询、搜索，限制工具权限范围

**小项目 B**：RAG + 工具混合智能体

* 用户问题 → 检索 → 若需要再调用业务 API（如库存/订单）→ 合成最终答复
* 指标：工具调用成功率、幻觉率（人工抽样）与响应时间

---

## 3️⃣ 可部署平台化（K8s/Knative/KServe）

* **推理服务**：vLLM/SGLang + **KServe**（支持多模型、多版本）
* **Serverless**：Knative autoscaling（并发/CPU/HPA），灰度/流量分配（Revision）
* **多租户**：按命名空间/域名隔离、限流与配额、Harbor 项目隔离、租户级密钥/向量库实例
* **CI/CD**：Tekton 流水线（git-clone → buildpacks → 单测 → 镜像 → 部署 KServe/Knative）
* **数据通道**：Kafka/NATS 做异步解析与向量入库；CDC（Debezium）同步结构化数据到向量库

**里程碑**：上线一个**多租户 RAG 服务**

* 控制面（Java/Go）：创建知识库、导入数据、配置模型路由
* 数据面：KServe 提供推理，应用面合成答案
* 观测与告警：P95 延迟、错误率、成本看板

---

## 4️⃣ 高阶能力（治理/评测/成本）

* 评测：离线评测集（问-答-证据）、在线 A/B、回放与对比
* 安全：PII 脱敏、越狱/提示注入防护、输出过滤（正则/规则+模型）
* 成本：模型路由（按质量/延迟/价格），缓存（语义缓存），分层检索
* 合规与审计：请求/响应审计日志、可追溯数据血缘

---

# 主流技术栈清单（按角色选型）

* **模型/推理**：OpenAI-compatible、vLLM、Ollama、LocalAI、KServe
* **Java**：Spring Boot 3、Spring AI、LangChain4j、gRPC、Micrometer、MapStruct、Testcontainers
* **Go**：chi/fiber、sashabaranov/go-openai、langchaingo、grpc-go、uber/fx or google/wire、zerolog
* **向量与检索**：Milvus / Qdrant / Weaviate / pgvector、Elasticsearch/OpenSearch（Hybrid）
* **数据处理**：Apache Tika、Airbyte/Beam（选学）、Kafka/NATS
* **平台与交付**：K8s、Knative、**KServe**、Tekton、Harbor、Buildpacks
* **观测与治理**：OpenTelemetry、Prometheus/Grafana、Loki/Jaeger/Tempo、OPA/Gatekeeper（策略）
* **安全**：Vault、KMS、WAF、PII 检测（可接 Python 服务）

---

# 练习与检查（帮你巩固）

* 用一句话区分：**“直连模型 API” vs “自托管推理 + KServe”**（你来回答）
* 画出你的**数据流**（上传→解析→向量化→检索→应答），标出观测指标（QPS、P95、命中率、成本）
* 给你的 RAG 增加\*\*“答案引用”**和**“信心分”\*\*，并在日志里落指标

---

# 两条落地路径（可选其一优先）

* **Java 优先**：Spring AI / LangChain4j 做应用 → KServe 托管开源模型 → Knative 托管业务服务
* **Go 优先**：chi + go-openai 快速 API 层（高并发、轻量）→ 用 Java 侧做复杂 RAG/工作流

---

# 下一步我怎么帮你

我可以把以上路线拆成**3\~4 个两周冲刺**的任务板（Issue 列表+验收标准），并给出每阶段的代码骨架（Java/Go 各一套）。

**先问你一个问题（只要选项）：**
你更想**先从哪条路线开始**？
A）Java 优先做一个“可观测的企业知识库 RAG（单租户）”
B）Go 优先做一个“高并发 Chat/工具调用 API 网关”，后续再接 RAG

选好后我就把对应的冲刺任务单和第一周的详细清单给你。
