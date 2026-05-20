# 处理器

## 学习目标

- 理解 Processor 在 SDK 管线中的作用。
- 掌握 Simple 与 Batch 处理器的差异。
- 理解 force flush 和 shutdown 对处理器的影响。

## 前置知识

- 队列与后台线程。
- 批处理。
- 资源释放。

## 核心概念

- Processor 位于 SDK 数据产生和 Exporter 之间。
- Processor 可以同步或异步处理遥测数据。
- Batch Processor 通常用于生产环境。

## 源码入口

- `sdk/trace/`
- `sdk/logs/`
- `sdk/metrics/`

## 关键类与接口

- `SpanProcessor`
- `SimpleSpanProcessor`
- `BatchSpanProcessor`
- `LogRecordProcessor`
- `BatchLogRecordProcessor`
- `MetricReader`

## 运行流程

- SDK 生成 Span 或 LogRecord。
- 调用 Processor 的开始、结束或 emit 钩子。
- Processor 决定立即导出或放入队列。
- flush 时处理积压数据。
- shutdown 时释放资源。

## 细化知识点

- Simple Processor 的同步导出。
- Batch Processor 的队列、调度和背压。
- 多 Processor 组合。
- dropped data 的处理。
- Processor 与 Exporter 的错误隔离。

## 常见问题

- 为什么生产环境通常不推荐 SimpleSpanProcessor？
- Batch 队列满了怎么办？
- shutdown 时如何保证尽量不丢数据？

## 实践任务

- 比较 Simple 和 Batch 处理器的导出时机。
- 阅读 BatchSpanProcessor 的配置项。
- 模拟 exporter 变慢时的队列行为。

## 延伸阅读

- OpenTelemetry SDK trace processor specification。

## 代码示例：Simple 与 Batch

```java
import io.opentelemetry.exporter.logging.LoggingSpanExporter;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import io.opentelemetry.sdk.trace.export.SimpleSpanProcessor;
import java.time.Duration;

class ProcessorExample {
  SdkTracerProvider simple() {
    return SdkTracerProvider.builder()
        // SimpleSpanProcessor 在 Span 结束时同步调用 exporter。
        // 适合测试和调试，不适合高吞吐生产路径。
        .addSpanProcessor(SimpleSpanProcessor.create(LoggingSpanExporter.create()))
        .build();
  }

  SdkTracerProvider batch() {
    return SdkTracerProvider.builder()
        .addSpanProcessor(
            BatchSpanProcessor.builder(LoggingSpanExporter.create())
                .setScheduleDelay(Duration.ofSeconds(5))
                .setMaxQueueSize(2048)
                .setMaxExportBatchSize(512)
                .build())
        .build();
  }
}
```

Batch 处理器通过队列把业务线程和导出线程隔离开。业务线程通常只负责把结束的 Span 放进队列，后台线程按批次导出。

## 处理器链路

```text
Span.start -> SpanProcessor.onStart
Span.end   -> SpanProcessor.onEnd
forceFlush -> Processor 尝试刷出队列
shutdown   -> Processor 停止接收并释放资源
```

如果 exporter 很慢，Batch 队列可能堆积。阅读 `BatchSpanProcessor` 时要关注：

- 队列满时如何处理。
- 批量大小如何控制。
- 定时导出如何触发。
- `forceFlush()` 和 `shutdown()` 如何等待后台工作。
