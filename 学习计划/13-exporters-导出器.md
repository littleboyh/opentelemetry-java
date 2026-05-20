# 导出器

## 学习目标

- 理解 Exporter 的职责和边界。
- 掌握 OTLP、Logging、Prometheus 等导出方式。
- 理解导出失败、重试和关闭流程。

## 前置知识

- HTTP、gRPC。
- protobuf。
- push 与 pull 数据模型。

## 核心概念

- Exporter 把 SDK 处理后的遥测数据发送到外部系统。
- OTLP 是 OpenTelemetry 的标准导出协议。
- 不同信号类型有各自的 Exporter 接口。

## 源码入口

- `exporters/`
- `exporters/otlp/`
- `exporters/logging/`
- `exporters/prometheus/`

## 关键类与接口

- `SpanExporter`
- `MetricExporter`
- `LogRecordExporter`
- `OtlpGrpcSpanExporter`
- `OtlpHttpSpanExporter`
- `PrometheusHttpServer`

## 运行流程

- Processor 或 Reader 调用 Exporter。
- Exporter 转换 SDK 数据到目标协议。
- 通过网络或本地输出发送数据。
- 返回 `CompletableResultCode` 表示成功或失败。
- shutdown 时关闭底层资源。

## 细化知识点

- OTLP gRPC 与 OTLP HTTP。
- exporter timeout。
- retry 策略。
- compression 和 headers。
- Prometheus pull 模型。
- logging exporter 的调试价值。

## 常见问题

- Exporter 失败会不会影响业务请求？
- OTLP HTTP 和 gRPC 应该如何选择？
- 为什么 Metrics 有 Prometheus pull exporter？

## 实践任务

- 配置 logging exporter 观察输出。
- 配置 OTLP exporter 指向本地 collector。
- 阅读 Exporter 如何转换 SpanData。

## 延伸阅读

- OTLP specification。
- OpenTelemetry Collector 文档。

## 代码示例：OTLP Exporter

```java
import io.opentelemetry.exporter.otlp.trace.OtlpGrpcSpanExporter;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import java.time.Duration;

class OtlpExporterExample {
  SdkTracerProvider tracerProvider() {
    OtlpGrpcSpanExporter exporter =
        OtlpGrpcSpanExporter.builder()
            .setEndpoint("http://localhost:4317")
            .setTimeout(Duration.ofSeconds(10))
            .build();

    return SdkTracerProvider.builder()
        .addSpanProcessor(BatchSpanProcessor.builder(exporter).build())
        .build();
  }
}
```

OTLP exporter 的职责是协议转换和发送，不应该包含业务逻辑。生产环境通常把数据发给 OpenTelemetry Collector，再由 Collector 做重试、路由、过滤、tail sampling 等后续处理。

## Exporter 接口边界

```text
SDK data object
  -> Exporter converts to protocol payload
  -> send through HTTP/gRPC/local output
  -> CompletableResultCode
```

`CompletableResultCode` 表示导出任务的异步结果。Processor 可以根据结果判断 flush 或 shutdown 是否完成，但业务代码通常不直接处理每次导出的结果。

## 阅读建议

- 先看 `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/export/SpanExporter.java`。
- 再看 OTLP exporter 的 builder，理解 endpoint、timeout、headers、compression。
- 最后看测试，确认失败和 shutdown 行为。
