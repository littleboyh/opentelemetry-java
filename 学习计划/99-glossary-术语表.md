# 术语表

## 学习目标

- 统一学习过程中使用的核心术语。
- 记录英文原词、中文译法和简短解释。
- 作为后续章节写作时的命名参考。

## 术语

| 英文 | 中文 | 说明 |
| --- | --- | --- |
| OpenTelemetry | 开放遥测 | 可观测性标准和工具集合。 |
| Trace | 链路 | 一次请求在分布式系统中的完整路径。 |
| Span | 跨度 | 链路中的一个操作单元。 |
| SpanContext | Span 上下文 | 可跨进程传播的 Trace 和 Span 标识信息。 |
| Context | 上下文 | 进程内传递当前状态的容器。 |
| Scope | 作用域 | 当前上下文生效的一段生命周期。 |
| Baggage | 行李 | 可跨服务传播的业务键值集合。 |
| Propagator | 传播器 | 负责注入和提取跨进程上下文。 |
| Attribute | 属性 | 遥测数据上的键值标签。 |
| Resource | 资源 | 产生遥测数据的实体描述。 |
| Instrumentation Scope | 插桩作用域 | 标识遥测数据来源库或组件的信息。 |
| Tracer | 链路追踪器 | 创建 Span 的 API 入口。 |
| Meter | 指标仪表入口 | 创建指标 Instrument 的 API 入口。 |
| Logger | 日志记录器 | 创建 LogRecord 的 API 入口。 |
| Exporter | 导出器 | 发送遥测数据到外部系统的组件。 |
| Processor | 处理器 | 位于 SDK 和 Exporter 之间的处理组件。 |
| Sampler | 采样器 | 决定 Span 是否采样的组件。 |
| MetricReader | 指标读取器 | 从 SDK 收集并导出指标数据的组件。 |
| OTLP | OpenTelemetry Protocol | OpenTelemetry 标准导出协议。 |
| Semantic Conventions | 语义约定 | 标准化属性名和事件命名的规则。 |

## 后续扩展方式

- 随章节扩写持续补充术语。
- 每个术语后续可以增加“常见误解”和“源码位置”。

## 易混术语对照

| 术语 A | 术语 B | 区别 |
| --- | --- | --- |
| API | SDK | API 是稳定调用契约；SDK 是具体处理和导出实现。 |
| Span | SpanContext | Span 是操作本身；SpanContext 是可传播的标识和采样状态。 |
| Context | Baggage | Context 是进程内上下文容器；Baggage 是存放在 Context 中并可跨服务传播的键值集合。 |
| Attribute | Baggage | Attribute 描述遥测数据；Baggage 描述需要跨服务传播的少量上下文。 |
| Resource | Instrumentation Scope | Resource 描述产生数据的实体；Instrumentation Scope 描述产生数据的库或组件。 |
| Processor | Exporter | Processor 处理 SDK 数据流；Exporter 把数据转换并发送到外部系统。 |
| MetricReader | MetricExporter | Reader 触发和组织采集；Exporter 负责发送采集结果。 |
| Head Sampling | Tail Sampling | Head sampling 在 Span 创建时决策；tail sampling 在看到更多链路数据后决策。 |

## 源码定位索引

| 主题 | 入口路径 |
| --- | --- |
| Context | `context/src/main/java/io/opentelemetry/context/Context.java` |
| Scope | `context/src/main/java/io/opentelemetry/context/Scope.java` |
| OpenTelemetry API | `api/all/src/main/java/io/opentelemetry/api/OpenTelemetry.java` |
| Global API | `api/all/src/main/java/io/opentelemetry/api/GlobalOpenTelemetry.java` |
| Trace API | `api/all/src/main/java/io/opentelemetry/api/trace/Tracer.java` |
| Trace SDK | `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/SdkTracerProvider.java` |
| Metrics SDK | `sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/SdkMeterProvider.java` |
| Logs SDK | `sdk/logs/src/main/java/io/opentelemetry/sdk/logs/SdkLoggerProvider.java` |
| Autoconfigure | `sdk-extensions/autoconfigure/src/main/java/io/opentelemetry/sdk/autoconfigure/AutoConfiguredOpenTelemetrySdk.java` |
| Testing | `sdk/testing/src/main/java/io/opentelemetry/sdk/testing/exporter/` |
