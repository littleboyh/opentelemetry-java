# 日志

## 学习目标

- 理解 OpenTelemetry Logs 的定位。
- 掌握 LogRecord、LoggerProvider、LogRecordProcessor 的关系。
- 理解日志与 Trace Context 的关联方式。

## 前置知识

- Java 日志框架基础。
- 结构化日志。
- Trace 与 Log 关联。

## 核心概念

- OpenTelemetry Logs 提供统一日志数据模型。
- LogRecord 可以携带 severity、body、attributes 和 context。
- 日志 SDK 负责处理和导出结构化日志。

## 源码入口

- `api/logs/`
- `sdk/logs/`
- `exporters/otlp/`

## 关键类与接口

- `Logger`
- `LoggerProvider`
- `LogRecordBuilder`
- `SdkLoggerProvider`
- `LogRecordProcessor`
- `LogRecordExporter`

## 运行流程

- 获取 OpenTelemetry Logger。
- 创建 LogRecord。
- 设置 body、severity、attributes 和 context。
- emit 日志。
- SDK 处理并导出。

## 细化知识点

- Log signal 的成熟度。
- 日志 body 与 attributes。
- Trace ID 和 Span ID 如何关联日志。
- 与 JUL、Logback、Log4j 等框架的关系。
- BatchLogRecordProcessor。

## 常见问题

- OpenTelemetry Logs 是否替代现有日志框架？
- 什么时候需要直接使用 Logs API？
- 日志如何关联当前 Span？

## 实践任务

- 写一个最小 LogRecord 示例。
- 阅读日志 SDK 的 Provider 初始化流程。
- 对比 SpanProcessor 和 LogRecordProcessor。

## 延伸阅读

- OpenTelemetry Logs specification。

## 代码示例：结构化 LogRecord

```java
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.api.logs.Logger;
import io.opentelemetry.api.logs.Severity;
import static io.opentelemetry.api.common.AttributeKey.stringKey;

class LogsExample {
  private final Logger logger;

  LogsExample(Logger logger) {
    this.logger = logger;
  }

  void emitPaymentFailed(String orderId) {
    logger
        .logRecordBuilder()
        .setSeverity(Severity.ERROR)
        .setBody("payment failed")
        .setAllAttributes(Attributes.of(stringKey("order.id"), orderId))
        // 如果当前 Context 中有 Span，SDK 可以把 trace id/span id 关联到日志。
        .emit();
  }
}
```

Logs API 的重点是把日志事件结构化，并保留与当前上下文的关系。它不一定替代应用已有日志框架，更常见的形态是通过 bridge 或 appender 把现有日志转换成 OpenTelemetry LogRecord。

## 源码阅读主线

1. `api/all/src/main/java/io/opentelemetry/api/logs/Logger.java`
2. `sdk/logs/src/main/java/io/opentelemetry/sdk/logs/SdkLoggerProvider.java`
3. `sdk/logs/src/main/java/io/opentelemetry/sdk/logs/LogRecordProcessor.java`
4. `sdk/logs/src/main/java/io/opentelemetry/sdk/logs/export/LogRecordExporter.java`

## 与 Trace 的关系

日志和链路追踪的关联通常来自当前 `Context`。如果业务代码在 Span 的 Scope 内写日志，LogRecord 就有机会带上 trace id 和 span id。排查“日志没有 trace id”时，先确认写日志的线程上是否存在当前 Span。
