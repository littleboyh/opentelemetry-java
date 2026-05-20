# 测试工具与验证

## 学习目标

- 掌握 OpenTelemetry Java 的测试辅助工具。
- 理解如何验证 Span、Metric、Log 输出。
- 建立学习源码时的最小验证方法。

## 前置知识

- JUnit。
- AssertJ。
- Gradle 测试任务。

## 核心概念

- 测试导出器可以在内存中收集遥测数据。
- SDK 测试通常需要显式 flush 或 shutdown。
- 验证遥测数据时要关注属性、上下文和时序。

## 源码入口

- `testing-internal/`
- `sdk/testing/`
- `integration-tests/`

## 关键类与接口

- `InMemorySpanExporter`
- `InMemoryMetricReader`
- `InMemoryLogRecordExporter`
- `OpenTelemetryRule` 如果当前版本包含相关测试工具。

## 运行流程

- 创建 SDK 和内存导出器。
- 执行业务或组件代码。
- 调用 flush 或 collect。
- 断言导出的数据内容。
- shutdown 清理资源。

## 细化知识点

- Trace 测试。
- Metrics 测试。
- Logs 测试。
- 时间、时钟和异步导出的处理。
- 测试隔离和全局状态清理。

## 常见问题

- 为什么测试里看不到导出的 Span？
- Metrics 为什么需要主动 collect？
- 使用 GlobalOpenTelemetry 的测试如何避免互相影响？

## 实践任务

- 写一个 InMemorySpanExporter 测试。
- 写一个 InMemoryMetricReader 测试。
- 阅读项目中的典型 SDK 测试。

## 延伸阅读

- `docs/knowledge/testing-patterns.md`

## 代码示例：InMemorySpanExporter

```java
import static org.assertj.core.api.Assertions.assertThat;

import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.testing.exporter.InMemorySpanExporter;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.SimpleSpanProcessor;
import org.junit.jupiter.api.Test;

class TraceTestExample {
  @Test
  void exportsSpan() {
    InMemorySpanExporter exporter = InMemorySpanExporter.create();
    SdkTracerProvider tracerProvider =
        SdkTracerProvider.builder()
            .addSpanProcessor(SimpleSpanProcessor.create(exporter))
            .build();
    OpenTelemetrySdk openTelemetry =
        OpenTelemetrySdk.builder().setTracerProvider(tracerProvider).build();

    Tracer tracer = openTelemetry.getTracer("test");
    tracer.spanBuilder("checkout").startSpan().end();

    assertThat(exporter.getFinishedSpanItems())
        .singleElement()
        .satisfies(span -> assertThat(span.getName()).isEqualTo("checkout"));

    tracerProvider.shutdown();
  }
}
```

测试里使用 `SimpleSpanProcessor` 可以减少异步等待。如果使用 `BatchSpanProcessor`，通常需要 `forceFlush()` 后再断言。

## 验证思路

- Trace：断言 Span 名称、属性、父子关系、状态、事件。
- Metrics：触发记录后调用 reader collect，再断言聚合结果。
- Logs：断言 severity、body、attributes、trace/span context。

测试全局实例时要特别小心。`GlobalOpenTelemetry` 是全局状态，测试之间如果没有清理，容易互相污染。
