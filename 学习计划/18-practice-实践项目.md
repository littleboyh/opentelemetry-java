# 实践项目

## 学习目标

- 通过小项目串联 OpenTelemetry Java 的核心能力。
- 把概念验证转换成可运行代码。
- 为每章补充实验依据。

## 前置知识

- Java 应用启动流程。
- HTTP 服务和客户端。
- Docker 或本地 OpenTelemetry Collector。

## 核心概念

- 学习 OpenTelemetry 最有效的方式是持续验证数据是否真的产生、传播和导出。
- 实践项目应从最小示例开始，再逐步增加 SDK 配置、传播、采样和导出。

## 源码入口

- `integration-tests/`
- `sdk/testing/`
- `exporters/`

## 关键类与接口

- 根据实践项目逐步补充。

## 运行流程

- 创建最小 Java 应用。
- 初始化 OpenTelemetry SDK。
- 生成 Trace、Metric、Log。
- 配置 OTLP 或 logging exporter。
- 接入 Collector 或内存测试工具验证输出。

## 细化知识点

- 最小手动埋点应用。
- HTTP 上下文传播实验。
- 自定义 Resource 和 Attribute。
- Trace 采样实验。
- Metrics View 实验。
- Log 与 Trace 关联实验。
- 自动配置实验。
- 自定义扩展实验。

## 常见问题

- 为什么后端看不到数据？
- Trace 断链如何排查？
- 指标数量为什么突然变多？
- 自动配置和手动配置冲突如何定位？

## 实践任务

- 项目一：只用 API 写 no-op 示例。
- 项目二：手动创建 SDK 并导出到 logging exporter。
- 项目三：接入本地 Collector。
- 项目四：模拟两个服务并验证上下文传播。
- 项目五：实现一个简单自定义 Exporter。

## 延伸阅读

- OpenTelemetry Collector 文档。
- OpenTelemetry Java examples。

## 项目一：最小 Trace 到日志导出

目标：手动创建 SDK，生成一个 Span，并通过 logging exporter 观察输出。

步骤：

1. 创建 `SdkTracerProvider`。
2. 注册 `BatchSpanProcessor`。
3. 使用 `LoggingSpanExporter`。
4. 创建 `Tracer` 并生成 Span。
5. 调用 `forceFlush()` 和 `shutdown()`。

关键代码：

```java
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.exporter.logging.LoggingSpanExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;

class PracticeTraceLogging {
  public static void main(String[] args) {
    SdkTracerProvider tracerProvider =
        SdkTracerProvider.builder()
            .addSpanProcessor(
                BatchSpanProcessor.builder(LoggingSpanExporter.create()).build())
            .build();

    OpenTelemetrySdk sdk =
        OpenTelemetrySdk.builder().setTracerProvider(tracerProvider).build();

    Tracer tracer = sdk.getTracer("practice");
    tracer.spanBuilder("practice-span").startSpan().end();

    tracerProvider.forceFlush();
    tracerProvider.shutdown();
  }
}
```

观察点：如果忘记 `end()`，Span 不会进入 Processor；如果使用 Batch 但不 flush 或 shutdown，程序退出前可能看不到输出。

## 项目二：两个服务的上下文传播

目标：模拟客户端和服务端，用 Map 代替 HTTP headers，验证 `traceparent` 注入和提取。

验收标准：

- headers 中出现 `traceparent`。
- 服务端 Span 的 parent 来自客户端提取的 Context。
- 两个 Span 具有相同 trace id。

## 项目三：Metrics View 实验

目标：记录 Histogram，配置不同 bucket，观察导出的聚合结果变化。

重点问题：

- 默认聚合是什么？
- View 如何匹配 Instrument？
- 属性过多时导出结果如何变化？

## 项目四：自动配置实验

目标：不用手动创建 SDK，通过环境变量配置 service name、exporter 和 endpoint。

建议先使用 logging exporter，再切换到 OTLP exporter。每切换一次配置，只改变一个变量，方便定位问题。
