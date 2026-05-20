# SDK 架构

## 学习目标

- 理解 SDK 在 OpenTelemetry Java 中负责什么。
- 掌握 Provider、Processor、Exporter 的协作方式。
- 建立 Trace、Metric、Log SDK 的共同模型。

## 前置知识

- Builder 模式。
- 生命周期管理。
- 批处理与后台线程。

## 核心概念

- SDK 是遥测数据的生产、处理、聚合和导出实现。
- Provider 是每类信号的入口。
- Processor 和 Exporter 负责数据处理与发送。

## 源码入口

- `sdk/`
- `sdk/common/`
- `sdk/trace/`
- `sdk/metrics/`
- `sdk/logs/`

## 关键类与接口

- `OpenTelemetrySdk`
- `SdkTracerProvider`
- `SdkMeterProvider`
- `SdkLoggerProvider`
- `CompletableResultCode`
- `Resource`

## 运行流程

- 使用 Builder 创建 SDK Provider。
- 注册 Processor、Exporter、Reader 或 View。
- 构造 `OpenTelemetrySdk`。
- 应用通过 API 生成数据。
- SDK 在 shutdown 或 force flush 时处理剩余数据。

## 细化知识点

- SDK Provider 生命周期。
- force flush 和 shutdown 的语义。
- shared state 和组件复用。
- clock、id generator、resource 等基础设施。
- SDK 与 API 的桥接方式。

## 常见问题

- 为什么 SDK Provider 需要关闭？
- `forceFlush()` 和 `shutdown()` 有什么区别？
- 多个 Exporter 如何组合？

## 实践任务

- 手动创建一个 `OpenTelemetrySdk`。
- 找出 Trace、Metric、Log Provider 的共同接口。
- 阅读 `CompletableResultCode` 的用法。

## 延伸阅读

- OpenTelemetry SDK specification。

## 代码示例：手动组装 SDK

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.exporter.logging.LoggingSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;

class SdkBootstrap {
  OpenTelemetry initOpenTelemetry() {
    SdkTracerProvider tracerProvider =
        SdkTracerProvider.builder()
            .setResource(Resource.getDefault())
            .addSpanProcessor(
                BatchSpanProcessor.builder(LoggingSpanExporter.create()).build())
            .build();

    return OpenTelemetrySdk.builder().setTracerProvider(tracerProvider).build();
  }
}
```

这段代码展示了 SDK 的核心组装关系：

- `SdkTracerProvider` 负责创建 SDK 版本的 `Tracer` 和 `Span`。
- `BatchSpanProcessor` 接收结束后的 Span，并批量交给 exporter。
- `LoggingSpanExporter` 把 Span 打到日志里，适合学习和调试。
- `OpenTelemetrySdk` 把各类 Provider 组合成 API 可使用的 `OpenTelemetry`。

## 生命周期重点

SDK 组件通常有资源需要释放，例如后台线程、队列、网络连接。应用关闭时应该调用：

```java
OpenTelemetrySdk sdk = OpenTelemetrySdk.builder().build();

// 尝试把缓冲区中的数据刷出去。
sdk.getSdkTracerProvider().forceFlush();

// 释放 Provider、Processor、Exporter 等资源。
sdk.getSdkTracerProvider().shutdown();
```

阅读 `SdkTracerProvider`、`SdkMeterProvider`、`SdkLoggerProvider` 时，可以对比它们的 `forceFlush()` 和 `shutdown()` 语义。
