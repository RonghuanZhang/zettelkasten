---
"type:": fleet-note
"title:": 20250604212027-Golang ZAP Logger
"id:": 20250604212044
"created:": 2025-06-04T21:20:44
url: 
tags:
  - fleet-note
  - golang
  - golang/zap
"processed:": false
"archived:": false
---
下面对这段 `logger` 包的代码从可用性、可维护性、可扩展性、安全性等多个角度做一个全面的评审，并给出一些优化建议。

```go
package logger

import (
	"go.uber.org/zap"
	"go.uber.org/zap/zapcore"
	"gopkg.in/natefinch/lumberjack.v2"
	"os"
	"time"
)

var Logger *zap.Logger

func InitLogger(logFile string) {
	// Encoder config for formatting
	encoderConfig := zapcore.EncoderConfig{
		TimeKey:        "time",
		LevelKey:       "level",
		NameKey:        "logger",
		CallerKey:      "caller",
		MessageKey:     "msg",
		StacktraceKey:  "stacktrace",
		LineEnding:     zapcore.DefaultLineEnding,
		EncodeLevel:    zapcore.CapitalColorLevelEncoder,
		EncodeTime:     zapcore.TimeEncoderOfLayout(time.RFC3339),
		EncodeDuration: zapcore.SecondsDurationEncoder,
		EncodeCaller:   zapcore.ShortCallerEncoder,
	}

	// Console output
	consoleEncoder := zapcore.NewConsoleEncoder(encoderConfig)
	consoleWriter := zapcore.Lock(os.Stdout)

	// File output with rolling
	fileWriter := zapcore.AddSync(&lumberjack.Logger{
		Filename:   logFile,
		MaxSize:    100, // MB
		MaxBackups: 10,
		MaxAge:     30, // days
		Compress:   true,
	})

	fileEncoder := zapcore.NewJSONEncoder(encoderConfig)

	core := zapcore.NewTee(
		zapcore.NewCore(consoleEncoder, consoleWriter, zapcore.InfoLevel),
		zapcore.NewCore(fileEncoder, fileWriter, zapcore.InfoLevel),
	)

	Logger = zap.New(core, zap.AddCaller(), zap.AddCallerSkip(1))
}

func Sync() {
	if Logger != nil {
		_ = Logger.Sync()
	}
}
```

---

## 1. 优点

1. **简单清晰的双输出设计**
    
    - 同时向控制台和文件输出，并且控制台采用带颜色的格式 (`CapitalColorLevelEncoder`)，文件采用 JSON 格式，满足大多数运维/开发场景。
        
2. **基于 `lumberjack` 实现文件滚动（Log Rotation）**
    
    - 限制了单个文件大小、备份数量和最大保留天数，并且启用了压缩，符合生产环境对日志管理的基本需求。
        
3. **全局 `Logger` 对象**
    
    - 通过 `var Logger *zap.Logger` 暴露给包外，调用方无须在各个文件中都传递 `*zap.Logger` 实例，使用起来非常方便。
        
4. **`Sync()` 方法**
    
    - 程序退出之前可以显式调用 `logger.Sync()` 将缓冲区里的日志强制 flush，防止丢失。
        

---

## 2. 需要注意和改进的地方

### 2.1 全局可变变量带来的隐患

- **问题**：  
    `var Logger *zap.Logger` 是一个包级变量，外部只要导入了 `logger` 包就能直接使用。
    
    - 如果测试、初始化顺序不当，或者有人提前拿了 `Logger` 去用，就可能出现空指针。
        
    - 后续如果想做热更新（动态调整日志级别）、或者多个子模块需要不同的日志配置（不同日志级别、不同输出路径），都比较麻烦。
        
- **优化建议**：
    
    1. **延迟初始化 & 完全控制初始化时机**
        
        - 将 `Logger` 设为私有变量，比如 `var globalLogger *zap.Logger`，并提供一个返回实例的方法：
            
            ```go
            var globalLogger *zap.Logger
            
            func Init(logFile string) error {
                if globalLogger != nil {
                    return errors.New("logger already initialized")
                }
                // …（保持 InitLogger 的逻辑，只是把 Logger 换成 globalLogger）…
                globalLogger = zap.New(core, zap.AddCaller(), zap.AddCallerSkip(1))
                return nil
            }
            
            func L() *zap.Logger {
                if globalLogger == nil {
                    panic("logger not initialized. call Init first")
                }
                return globalLogger
            }
            
            func Sync() {
                if globalLogger != nil {
                    _ = globalLogger.Sync()
                }
            }
            ```
            
        - 这样一方面避免「零值时调用空指针」的问题，另一方面调用方也更显式：必须先 `logger.Init(...)`，才能拿到 `logger.L()`。
            
    2. **可选多实例**
        
        - 如果对日志分割（例如不同业务模块分别输出到不同文件）有需求，可以让 `Init` 返回一个 `*zap.Logger` 实例：
            
            ```go
            func NewLogger(config LoggerConfig) (*zap.Logger, error) { … }
            ```
            
        - 调用方自行持有并在需要时传递，让包内部不再维护「全局变量」。
            

---

### 2.2 日志级别的灵活配置

- **问题**：  
    当前写死了两个 `NewCore(..., zapcore.InfoLevel)`，只能输出 `Info` 及以上级别。
    
    - 如果想在开发环境里调成 `Debug`，或生产环境调成 `Error`，就要修改源码或给一个全局变量开关。
        
- **优化建议**：
    
    1. **抽象成 `AtomicLevel`**
        
        - 利用 `zap.NewAtomicLevelAt(zapcore.InfoLevel)`，把 level 当作一个可动态修改的值：
            
            ```go
            level := zap.NewAtomicLevelAt(zapcore.InfoLevel)
            core := zapcore.NewTee(
                zapcore.NewCore(consoleEncoder, consoleWriter, level),
                zapcore.NewCore(fileEncoder, fileWriter, level),
            )
            Logger = zap.New(core, zap.AddCaller(), zap.AddCallerSkip(1))
            ```
            
        - 然后可在运行时通过 `level.SetLevel(zapcore.DebugLevel)` 动态调整。
            
    2. **从外部传入配置**
        
        - 在 `InitLogger` 里增加一个参数 `logLevel zapcore.Level`：
            
            ```go
            func InitLogger(logFile string, logLevel zapcore.Level) { … }
            ```
            
        - 或者先读取环境变量（`os.Getenv("LOG_LEVEL")`）来决定 `zapcore.Level`，让代码更通用。
            

---

### 2.3 `AddCallerSkip(1)` 的合理性

- **问题**：  
    你在创建 `zap.New(...)` 时，传了 `zap.AddCaller()` 和 `zap.AddCallerSkip(1)`。
    
    - `AddCaller()` 打印调用者（文件名 + 行号），默认会把 `InitLogger`、`Sync`、或任何直接调用 `Logger.Info(…)` 的层级都打出来。
        
    - `AddCallerSkip(1)` 让调用者跳过一层。通常如果你再写一个封装函数（比如 `func Info(args …)`），就需要加 skip。
        
- **优化建议**：
    
    - 如果没有在外层对 `Logger` 做二次封装，就可以不使用 `AddCallerSkip(1)`。否则要根据自己的调用链深度来设定 skip 值。
        
    - 可以先在开发环境打印几条日志，确认显示的文件名、行号是你想要的。如果显示不对再调整。
        

---

### 2.4 对 `lumberjack.Logger` 的维护

- **问题**：
    
    - 虽然 `lumberjack.Logger` 本身并不需要手动 `Close()`，但在某些场景下如果想彻底释放资源（尤其是 Windows 系统下），显式关闭也更安全。
        
- **优化建议**：
    
    - 如果你改成自己创建 `lumberjack.Logger`，可以把它当成一个字段保存在某个结构体里，然后在程序退出时调用 `Close()`（`lumberjack.Logger` 实现了 `io.WriteCloser`）。
        
    - 当然大多数场景下依赖 `Sync()` 已经可以让 Zap flush 缓冲区，操作系统会在进程退出时释放文件句柄。
        

---

### 2.5 错误处理与可扩展性

- **问题**：
    
    - `InitLogger` 函数本身没有返回 `error`，如果创建 `zap.NewCore` 过程中、或者 `lumberjack.Logger` 初始化时出现问题（例如文件路径不可写），你拿不到任何错误。
        
    - 如果后续想要支持「日志分隔到多个目录」、「本地调试时不输出到文件、仅输出到控制台」等需求，就要改函数签名。
        
- **优化建议**：
    
    1. **将 `InitLogger` 改为返回 `error`**，并在内部对可能出现的错误（比如打开文件、创建目录失败）做捕获：
        
        ```go
        func InitLogger(logFile string) error {
            // 尝试创建日志目录
            if err := os.MkdirAll(filepath.Dir(logFile), 0755); err != nil {
                return fmt.Errorf("create log dir failed: %w", err)
            }
            // … 其它逻辑 …
            return nil
        }
        ```
        
    2. **支持更灵活的配置接口**
        
        - 定义一个 `struct LoggerConfig { FilePath string; MaxSize, MaxBackups, MaxAge int; Compress bool; Level zapcore.Level; ToConsole bool; ToFile bool }`
            
        - 然后写一个 `NewLogger(cfg LoggerConfig) (*zap.Logger, error)`，让调用方自行决定是否要控制台输出、要不要文件输出。
            

---

### 2.6 EncoderConfig 细节

- **问题**：
    
    - 你把 `EncodeLevel` 设置为 `CapitalColorLevelEncoder`，这在控制台看很清晰，但如果把 JSON 写进文件，可能会带着 ANSI 颜色码。幸好你给 JSON 用的是 `NewJSONEncoder(encoderConfig)`，它不会带颜色；但是请确保在实际运行时 JSON 文件不会混进颜色。
        
    - `EncodeTime: zapcore.TimeEncoderOfLayout(time.RFC3339)`：这个非常标准，但要注意有些团队喜欢使用纳秒级时间戳或者 UNIX 毫秒，需根据需求调整。
        
- **优化建议**：
    
    - 明确把「控制台用带颜色的 EncoderConfig」和「文件用纯 JSON 的 EncoderConfig」分开。可以把两套 `EncoderConfig` 明确写成两个常量或两个变量，方便未来扩展。
        
    - 如果要兼容外部日志聚合系统，建议在 JSON 字段里加一个 `service` 或 `app` 字段，用来标识日志来源：
        
        ```go
        encoderConfig := zapcore.EncoderConfig{
            TimeKey:        "time",
            LevelKey:       "level",
            NameKey:        "service", // 原来叫 logger，改成 service 更贴合一些
            // …
        }
        ```
        

---

### 2.7 包名与导出 API 风格

- **问题**：
    
    - 包名 `logger` 语义没问题，但暴露出来的 `Logger` 变量直接大写导出，意味着外部代码可以随意改成 `logger.Logger = nil`，或者直接赋一个新的实例。
        
    - 如果想隐藏内部实现，可以只导出少数方法（比如 `Info`, `Error`, `Init`），而把底层 `*zap.Logger` 保留为私有。
        
- **优化建议**：
    
    1. **隐藏全局变量**
        
        - 将 `var Logger *zap.Logger` 改成 `var globalLogger *zap.Logger`（小写），并提供类似下面的一组包装函数：
            
            ```go
            func Init(logFile string) error { … }
            func Info(msg string, fields ...zap.Field) {
                L().Info(msg, fields...)
            }
            func Error(msg string, fields ...zap.Field) {
                L().Error(msg, fields...)
            }
            // … 以及 Debug、Warn、DPanic、Panic、Fatal 等方法  
            ```
            
        - 这样使用方只能通过 `logger.Info(...)`、`logger.Error(...)` 去记录日志，无法直接对 `*zap.Logger` 做不安全的操作。
            
    2. **附加上下文信息**
        
        - 日志里常常需要添加服务名、版本号、环境变量（dev/prod）、主机 IP 等元信息。
            
        - 可以在 `Init` 时调用：
            
            ```go
            Logger = zap.New(core,
                zap.AddCaller(),
                zap.AddCallerSkip(1),
                zap.Fields(
                    zap.String("service", cfg.ServiceName),
                    zap.String("env", cfg.Env),
                    zap.String("hostname", hostname),
                ),
            )
            ```
            

---

## 3. 重构示例

下面给出一个更为灵活可配置的重构思路，仅作参考（不一定要完全照搬）：

```go
package logger

import (
    "errors"
    "fmt"
    "go.uber.org/zap"
    "go.uber.org/zap/zapcore"
    "gopkg.in/natefinch/lumberjack.v2"
    "os"
    "path/filepath"
)

// LoggerConfig 封装了日志系统的所有可配置项
type LoggerConfig struct {
    FilePath    string          // 日志文件路径。如果为空，则仅输出到控制台
    MaxSize     int             // 单个日志文件最大尺寸（MB），仅在 FilePath 不为空时生效
    MaxBackups  int             // 日志文件备份数量
    MaxAge      int             // 日志文件保留天数
    Compress    bool            // 是否压缩归档
    Level       zapcore.Level   // 日志级别
    WithConsole bool            // 是否输出到控制台
    WithCaller  bool            // 是否显示 caller：文件名+行号
    ServiceName string          // 服务名，用于日志字段中区分不同服务
    Env         string          // 环境（dev/test/prod）
}

var globalLogger *zap.Logger

// Init 初始化全局日志
func Init(cfg LoggerConfig) error {
    if globalLogger != nil {
        return errors.New("logger已初始化")
    }

    // 如果指定了文件路径，确保目录存在
    if cfg.FilePath != "" {
        dir := filepath.Dir(cfg.FilePath)
        if err := os.MkdirAll(dir, 0755); err != nil {
            return fmt.Errorf("创建日志目录失败: %w", err)
        }
    }

    // 构造 EncoderConfig
    encoderConfig := zapcore.EncoderConfig{
        TimeKey:        "time",
        LevelKey:       "level",
        NameKey:        "service",
        CallerKey:      "caller",
        MessageKey:     "msg",
        StacktraceKey:  "stacktrace",
        LineEnding:     zapcore.DefaultLineEnding,
        EncodeLevel:    zapcore.CapitalColorLevelEncoder,
        EncodeTime:     zapcore.TimeEncoderOfLayout(time.RFC3339),
        EncodeDuration: zapcore.SecondsDurationEncoder,
        EncodeCaller:   zapcore.ShortCallerEncoder,
    }

    var cores []zapcore.Core

    // 控制台输出
    if cfg.WithConsole {
        consoleEncoder := zapcore.NewConsoleEncoder(encoderConfig)
        consoleWriter := zapcore.Lock(os.Stdout)
        cores = append(cores, zapcore.NewCore(consoleEncoder, consoleWriter, cfg.Level))
    }

    // 文件输出
    if cfg.FilePath != "" {
        lumberjackLogger := &lumberjack.Logger{
            Filename:   cfg.FilePath,
            MaxSize:    cfg.MaxSize,
            MaxBackups: cfg.MaxBackups,
            MaxAge:     cfg.MaxAge,
            Compress:   cfg.Compress,
        }
        writer := zapcore.AddSync(lumberjackLogger)
        fileEncoder := zapcore.NewJSONEncoder(encoderConfig)
        cores = append(cores, zapcore.NewCore(fileEncoder, writer, cfg.Level))
    }

    if len(cores) == 0 {
        return errors.New("至少需要指定控制台输出或文件输出")
    }

    // 使用 NewTee 合并 Core
    atomicLevel := zap.NewAtomicLevelAt(cfg.Level)
    tee := zapcore.NewTee(cores...)
    opts := []zap.Option{zap.AddCaller()} // 默认显示 caller
    if !cfg.WithCaller {
        opts = []zap.Option{}
    }
    // 添加全局字段：service、env
    opts = append(opts, zap.Fields(
        zap.String("service", cfg.ServiceName),
        zap.String("env", cfg.Env),
    ))

    globalLogger = zap.New(tee, opts...)

    return nil
}

// L 返回全局 *zap.Logger，必须在 Init 之后调用
func L() *zap.Logger {
    if globalLogger == nil {
        panic("logger 未初始化，请先调用 Init")
    }
    return globalLogger
}

// Sync flush 缓冲区
func Sync() {
    if globalLogger != nil {
        _ = globalLogger.Sync()
    }
}
```

- **亮点**：
    
    1. 将所有配置都用一个 `LoggerConfig` 结构体封装，让调用方只需关心自己想配置哪些字段。
        
    2. 支持「仅控制台」「仅文件」「两者都要」的灵活组合。
        
    3. 引入 `zap.NewAtomicLevelAt`，可以在以后动态调整日志级别。
        
    4. 封装了全局变量访问，只能通过 `logger.L()` 拿到 `*zap.Logger`，避免外部直接乱改。
        
    5. 在日志中统一添加了 `service`、`env` 字段，方便后续做检索和过滤。
        

---

## 4. 其他细节与最佳实践

1. **本地开发与生产环境区分**
    
    - 通常在本地开发时想要看到 `Debug` 甚至 `DPanic` 级别；在生产环境为了性能则会使用 `Info` 或更高等级。可以通过读取环境变量（如 `LOG_LEVEL=debug`）来决定 `LoggerConfig.Level`。
        
2. **Json vs Console**
    
    - 如果日志要消费给 Elasticsearch/Kibana 之类的系统，JSON 格式必不可少；但开发人员看日志还是更喜欢彩色控制台格式。将两者分开写更清晰。
        
3. **调用链深度（CallerSkip）**
    
    - 如果你在项目中再做了一层日志封装，比如 `func Info(ctx context.Context, msg string) { L().Info(msg, zap.String("traceID", TraceID(ctx))) }`，要调整 CallerSkip = 2 或更多，保证打印出来的文件名是业务代码，而不是这个包装函数。
        
4. **灾难恢复与降级**
    
    - 如果日志目录写满、或者打开日志文件失败，应考虑一种「降级策略」，比如退回到仅控制台输出，或者将日志记录到 STDERR 并退出服务。
        
5. **性能**
    
    - `zap` 本身是高速日志库，用二进制 JSON 序列化比标准库的 `log` 要快得多。但调用 `Sync()` 会产生 I/O 阻塞，一般放在服务优雅停止阶段再调用。线上不要频繁调用。
        
6. **测试**
    
    - 可以在单元测试时用一个「只输出到内存 buffer」的 Writer，或直接 `zap.NewNop()`，让单元测试不会生成文件。
        

---

## 5. 总结与行动项

- **如果对现有代码改动最小化**，可以：
    
    1. 检查 `AddCallerSkip` 是否确实要跳 1 层；
        
    2. 把日志级别改成 `AtomicLevel`，并从环境变量里读取；
        
    3. 最好让 `InitLogger` 返回 `error`。
        
- **如果准备做一次重构**，可以参考上面示例，把配置抽象成 `LoggerConfig`，让调用方自行选择是否输出到控制台、输出到文件，以及日志级别、切分策略等。外部不再直接拿到 `*zap.Logger`，而是通过 `logger.L()` 这种方式获取，保证调用方使用过程中不容易误操作。
    
- **最后，千万要在程序退出前调用 `logger.Sync()`**，避免日志丢失。如果你的程序是 HTTP 服务，可以在优雅关机逻辑里执行：
    
    ```go
    quit := make(chan os.Signal, 1)
    signal.Notify(quit, syscall.SIGINT, syscall.SIGTERM)
    <-quit
    logger.Sync()
    ```
    

以上建议均基于生产级别的 Go 项目实践以及社区常见模式，希望对你优化这份 logger 代码有所帮助。如果有其他细节或具体问题，欢迎随时讨论！

# Reference