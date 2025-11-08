---
"type:": fleet-note
"title:": 20251030162724-Gin OpenTelemetry Integration
id:: 20251030162733  # 唯一 ID，基于创建时间确保全局唯一
created:: 2025-10-30T16:27:33  # 创建时间（ISO 格式）
url: 
tags:
  - fleet-note
"processed:": false
"archived:": false
---
原因：你没有初始化 OpenTelemetry 的 SDK，全局还是默认的 no-op TracerProvider。`otelgin` 中间件会创建“非记录型”span，此时 `SpanContext` 无效，所以 `trace.SpanFromContext(...).SpanContext().IsValid()` 返回 `false`。OpenTelemetry 规范与实现都明确：只有包含有效 TraceID 和 SpanID 的 `SpanContext` 才是 “valid”。未配置 SDK 时使用的是 no-op 实现，产生的 `SpanContext` 无效。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/otel/trace?utm_source=chatgpt.com "trace package - go.opentelemetry.io/otel/trace"))

怎么修：

1. 初始化 TracerProvider 与 Propagator（最小可用示例）
    

```go
import (
  "context"
  "log"
  "go.opentelemetry.io/otel"
  "go.opentelemetry.io/otel/propagation"
  "go.opentelemetry.io/otel/exporters/stdout/stdouttrace"
  sdktrace "go.opentelemetry.io/otel/sdk/trace"
  "go.opentelemetry.io/otel/sdk/resource"
  semconv "go.opentelemetry.io/otel/semconv/v1.26.0"
)

func initOTel() func(context.Context) error {
  // 1) 导出器：示例用 stdout。生产建议 OTLP -> Collector
  exp, err := stdouttrace.New(stdouttrace.WithPrettyPrint())
  if err != nil { log.Fatal(err) }

  // 2) 资源：设置服务名
  res, err := resource.New(
    context.Background(),
    resource.WithAttributes(semconv.ServiceNameKey.String("your-service-name")),
  )
  if err != nil { log.Fatal(err) }

  // 3) TracerProvider：批处理 + 资源
  tp := sdktrace.NewTracerProvider(
    sdktrace.WithBatcher(exp),
    sdktrace.WithResource(res),
  )

  // 4) 设置全局 TracerProvider 和 TextMap Propagator
  otel.SetTracerProvider(tp)
  otel.SetTextMapPropagator(
    propagation.NewCompositeTextMapPropagator(propagation.TraceContext{}, propagation.Baggage{}),
  )
  return tp.Shutdown
}
```

在 `main()` 最前面调用：

```go
shutdown := initOTel()
defer shutdown(context.Background())
```

参考：Go 入门与 SDK 初始化、Propagator 说明。([OpenTelemetry](https://opentelemetry.io/docs/languages/go/getting-started/?utm_source=chatgpt.com "Getting Started"))

2. `otelgin` 放在最前即可。你已正确：`r.Use(otelgin.Middleware("your-service-name"))`。另外确保服务名来自 Resource，而不是依赖中间件字符串参数的历史行为。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin?utm_source=chatgpt.com "otelgin package - go.opentelemetry.io/contrib/ ..."))
    
3. `ginzap` 中读取 Trace/Span：  
    `gin-contrib/zap` 官方示例就是从 `c.Request.Context()` 里取 TraceID/SpanID。只要 1) 初始化了 SDK 或 2) 上游带 `traceparent`，`IsValid()` 就会为真。([GitHub](https://github.com/gin-contrib/zap?utm_source=chatgpt.com "gin-contrib/zap: Alternative logging through zap"))
    
4. 与上游 Spring Boot 协同：  
    确保上游开启 OTel，并通过 W3C Trace Context 头 `traceparent` 传递；本服务已设置 `TraceContext` Propagator 会自动提取并续链。([OpenTelemetry](https://opentelemetry.io/docs/concepts/context-propagation/?utm_source=chatgpt.com "Context propagation"))
    

快速验证步骤：

- 不接上游也能看到有效 TraceID：因为本服务会本地创建根 span（前提是已设置 SDK）。此时 `IsValid()` 应变为 `true`。([OpenTelemetry](https://opentelemetry.io/docs/languages/go/getting-started/?utm_source=chatgpt.com "Getting Started"))
    

补充说明：

- `SpanContext.IsValid()` 判定标准是 TraceID 与 SpanID 均非零。默认 no-op 提供者下创建的 span 为非记录型，`SpanContext` 无效。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/otel/trace?utm_source=chatgpt.com "trace package - go.opentelemetry.io/otel/trace"))
    

以上变更完成后，你的 `Context` 回调里这段将返回有效的 `trace_id` 与 `span_id`：

```go
sc := trace.SpanFromContext(c.Request.Context()).SpanContext()
if sc.IsValid() {
  fields = append(fields,
    zap.String("trace_id", sc.TraceID().String()),
    zap.String("span_id", sc.SpanID().String()),
  )
}
```

参考信息：

- OpenTelemetry Go `trace.SpanContext` 与 `IsValid` 定义。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/otel/trace?utm_source=chatgpt.com "trace package - go.opentelemetry.io/otel/trace"))
    
- `otelgin` 官方包与示例。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/contrib/instrumentation/github.com/gin-gonic/gin/otelgin?utm_source=chatgpt.com "otelgin package - go.opentelemetry.io/contrib/ ..."))
    
- Go 入门与全局 TracerProvider 初始化。([OpenTelemetry](https://opentelemetry.io/docs/languages/go/getting-started/?utm_source=chatgpt.com "Getting Started"))
    
- 上下文传播概念与 `TraceContext`。([OpenTelemetry](https://opentelemetry.io/docs/concepts/context-propagation/?utm_source=chatgpt.com "Context propagation"))
    
- no-op 提供者说明。([Go Packages](https://pkg.go.dev/go.opentelemetry.io/otel/trace%40v1.38.0/noop?utm_source=chatgpt.com "noop package - go.opentelemetry.io/otel/trace/noop"))
    
- `gin-contrib/zap` 自定义字段示例。([GitHub](https://github.com/gin-contrib/zap?utm_source=chatgpt.com "gin-contrib/zap: Alternative logging through zap"))

# Reference