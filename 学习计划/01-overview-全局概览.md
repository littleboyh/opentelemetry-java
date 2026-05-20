# 全局概览

## 学习目标

- 理解 OpenTelemetry Java 的整体架构。
- 区分 API、SDK、Exporter、Extension、Testing 等模块。
- 建立后续章节的全局地图。

## 前置知识

- Java 模块与包组织。
- 可观测性三大信号：Traces、Metrics、Logs。
- 服务间调用与分布式上下文。

## 核心概念

- API 定义用户可调用的抽象。
- SDK 提供遥测数据的具体实现、处理和生命周期管理。
- Exporter 负责把遥测数据发送到外部系统。
- Context 负责在调用链中保存和传递当前状态。

## 源码入口

- `api/`
- `context/`
- `sdk/`
- `exporters/`
- `extensions/`
- `sdk-extensions/`
- `testing-internal/`

## 关键类与接口

- `OpenTelemetry`
- `GlobalOpenTelemetry`
- `Context`
- `SdkTracerProvider`
- `SdkMeterProvider`
- `SdkLoggerProvider`

## 运行流程

- 应用初始化 OpenTelemetry。
- 获取 Tracer、Meter 或 Logger。
- 业务代码生成遥测数据。
- SDK 处理数据并调用 Exporter。
- 后端系统接收和展示数据。

## 细化知识点

- API 与 SDK 的依赖方向。
- 全局实例与显式传参两种使用方式。
- stable、alpha、internal 包的边界。
- 三大信号共享的基础设施。
- Gradle 多模块如何反映组件边界。

## 常见问题

- `api` 模块为什么不直接导出数据？
- 什么情况下需要直接依赖 `sdk`？
- `internal` 包里的代码能不能被外部使用？

## 实践任务

- 根据顶层目录整理一张组件关系表。
- 找出 Trace、Metric、Log 三条链路共享的抽象。
- 运行一次项目测试或编译，确认本地环境可用。

## 延伸阅读

- `docs/knowledge/README.md`
- `VERSIONING.md`

## 一张全局心智图

OpenTelemetry Java 可以按职责分成五层：

```text
Application / Library
  |
  v
API: OpenTelemetry, Tracer, Meter, Logger, Context
  |
  v
SDK: SdkTracerProvider, SdkMeterProvider, SdkLoggerProvider
  |
  v
Pipeline: Sampler, Processor, Reader, View, Aggregation
  |
  v
Exporter: OTLP, Logging, Prometheus, InMemory
```

理解这张图后，很多源码路径就不难判断：

- 业务调用入口在 `api/`。
- 当前上下文在 `context/`。
- 具体数据处理在 `sdk/`。
- 协议转换和发送在 `exporters/`。
- 环境变量、SPI 和自动组装在 `sdk-extensions/autoconfigure/`。

## 最小使用示例

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;

class CheckoutService {
  private final Tracer tracer;

  CheckoutService(OpenTelemetry openTelemetry) {
    // instrumentation scope name 通常使用库名或组件名。
    this.tracer = openTelemetry.getTracer("example-checkout");
  }

  void checkout() {
    Span span = tracer.spanBuilder("checkout").startSpan();

    // makeCurrent() 把 Span 放入当前 Context。
    // try-with-resources 确保 Scope 关闭，避免上下文泄漏到后续请求。
    try (Scope ignored = span.makeCurrent()) {
      span.setAttribute("cart.items", 3);
      charge();
    } catch (Throwable t) {
      span.recordException(t);
      throw t;
    } finally {
      // Span 必须结束，否则 SDK 不会把它交给 Processor/Exporter。
      span.end();
    }
  }

  private void charge() {}
}
```

这段代码只依赖 API。是否真的导出数据，取决于应用启动时是否注册了 SDK。

## 完整使用示例：从初始化到导出

下面的示例把全局概览里的关键组件串在一起：应用启动时创建 SDK，业务代码通过 API 记录 Trace、Metric 和 Log，最后显式 flush 和 shutdown。为了便于本地学习，示例使用 logging exporter 和标准输出，不依赖外部后端。

```java
import static io.opentelemetry.api.common.AttributeKey.longKey;
import static io.opentelemetry.api.common.AttributeKey.stringKey;

import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.api.logs.Logger;
import io.opentelemetry.api.logs.Severity;
import io.opentelemetry.api.metrics.DoubleHistogram;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.Meter;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;
import io.opentelemetry.exporter.logging.LoggingMetricExporter;
import io.opentelemetry.exporter.logging.LoggingSpanExporter;
import io.opentelemetry.exporter.logging.SystemOutLogRecordExporter;
import io.opentelemetry.sdk.OpenTelemetrySdk;
import io.opentelemetry.sdk.logs.SdkLoggerProvider;
import io.opentelemetry.sdk.logs.export.SimpleLogRecordProcessor;
import io.opentelemetry.sdk.metrics.SdkMeterProvider;
import io.opentelemetry.sdk.metrics.export.PeriodicMetricReader;
import io.opentelemetry.sdk.resources.Resource;
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.export.BatchSpanProcessor;
import java.time.Duration;
import java.util.concurrent.TimeUnit;

public final class CompleteOpenTelemetryExample {
  public static void main(String[] args) {
    Resource resource =
        Resource.getDefault()
            .merge(
                Resource.create(
                    Attributes.of(
                        stringKey("service.name"), "checkout-service",
                        stringKey("service.version"), "1.0.0")));

    // Trace 管线：Span 结束后进入 BatchSpanProcessor，再由 LoggingSpanExporter 输出。
    SdkTracerProvider tracerProvider =
        SdkTracerProvider.builder()
            .setResource(resource)
            .addSpanProcessor(
                BatchSpanProcessor.builder(LoggingSpanExporter.create()).build())
            .build();

    // Metrics 管线：业务代码记录 measurement，MetricReader 周期性收集聚合结果。
    SdkMeterProvider meterProvider =
        SdkMeterProvider.builder()
            .setResource(resource)
            .registerMetricReader(
                PeriodicMetricReader.builder(LoggingMetricExporter.create())
                    .setInterval(Duration.ofSeconds(5))
                    .build())
            .build();

    // Logs 管线：LogRecord emit 后进入 LogRecordProcessor，再由 exporter 输出。
    // 这里用 SimpleLogRecordProcessor 是为了示例直接可见；生产环境通常使用批处理器。
    SdkLoggerProvider loggerProvider =
        SdkLoggerProvider.builder()
            .setResource(resource)
            .addLogRecordProcessor(
                SimpleLogRecordProcessor.create(SystemOutLogRecordExporter.create()))
            .build();

    OpenTelemetrySdk openTelemetry =
        OpenTelemetrySdk.builder()
            .setTracerProvider(tracerProvider)
            .setMeterProvider(meterProvider)
            .setLoggerProvider(loggerProvider)
            .build();

    try {
      CheckoutWorkflow workflow = new CheckoutWorkflow(openTelemetry);
      workflow.checkout("order-1001", 3);
    } finally {
      // 学习示例里显式 flush，方便在进程退出前看到 BatchSpanProcessor 和 MetricReader 的输出。
      tracerProvider.forceFlush().join(10, TimeUnit.SECONDS);
      meterProvider.forceFlush().join(10, TimeUnit.SECONDS);
      loggerProvider.forceFlush().join(10, TimeUnit.SECONDS);

      // 关闭顺序不必死记，重点是应用退出前要释放 SDK 组件持有的线程和 exporter 资源。
      tracerProvider.shutdown().join(10, TimeUnit.SECONDS);
      meterProvider.shutdown().join(10, TimeUnit.SECONDS);
      loggerProvider.shutdown().join(10, TimeUnit.SECONDS);
    }
  }

  private static final class CheckoutWorkflow {
    private final Tracer tracer;
    private final LongCounter checkoutCounter;
    private final DoubleHistogram checkoutLatency;
    private final Logger logger;

    CheckoutWorkflow(OpenTelemetry openTelemetry) {
      // instrumentation scope name 标识“谁产生了这些遥测数据”。
      this.tracer = openTelemetry.getTracer("example.checkout");

      Meter meter = openTelemetry.getMeter("example.checkout");
      this.checkoutCounter =
          meter
              .counterBuilder("checkout.requests")
              .setDescription("Number of checkout requests")
              .setUnit("{request}")
              .build();
      this.checkoutLatency =
          meter
              .histogramBuilder("checkout.latency")
              .setDescription("Checkout latency")
              .setUnit("ms")
              .build();

      // Logs bridge API 的定位是桥接结构化日志，不是替代应用日志框架。
      this.logger = openTelemetry.getLogsBridge().get("example.checkout");
    }

    void checkout(String orderId, long itemCount) {
      long startNanos = System.nanoTime();
      Span span = tracer.spanBuilder("CheckoutWorkflow.checkout").startSpan();

      try (Scope ignored = span.makeCurrent()) {
        span.setAttribute("order.id", orderId);
        span.setAttribute("cart.items", itemCount);

        logger
            .logRecordBuilder()
            .setSeverity(Severity.INFO)
            .setBody("checkout started")
            .setAttribute(stringKey("order.id"), orderId)
            .emit();

        charge(orderId);

        checkoutCounter.add(1, Attributes.of(stringKey("checkout.result"), "success"));
        span.setStatus(StatusCode.OK);
      } catch (RuntimeException e) {
        checkoutCounter.add(1, Attributes.of(stringKey("checkout.result"), "error"));
        span.recordException(e);
        span.setStatus(StatusCode.ERROR, e.getMessage());

        logger
            .logRecordBuilder()
            .setSeverity(Severity.ERROR)
            .setBody("checkout failed")
            .setAttribute(stringKey("order.id"), orderId)
            .emit();
        throw e;
      } finally {
        long latencyMillis = Duration.ofNanos(System.nanoTime() - startNanos).toMillis();
        checkoutLatency.record(
            latencyMillis, Attributes.of(longKey("cart.items"), itemCount));

        // end() 是 Trace 管线的关键边界：Span 到这里才会进入 Processor。
        span.end();
      }
    }

    private void charge(String orderId) {
      Span span = tracer.spanBuilder("PaymentClient.charge").startSpan();
      try (Scope ignored = span.makeCurrent()) {
        span.setAttribute("order.id", orderId);
        span.setAttribute("payment.provider", "demo");
      } finally {
        span.end();
      }
    }
  }
}
```

这个完整示例可以按下面的顺序阅读：

1. `Resource` 先定义服务级属性，例如 `service.name`。
2. `SdkTracerProvider`、`SdkMeterProvider`、`SdkLoggerProvider` 分别组装三类信号的 SDK 管线。
3. `OpenTelemetrySdk` 把三类 Provider 包装成统一的 `OpenTelemetry` 入口。
4. 业务代码只拿 `Tracer`、`Meter`、`Logger`，不直接依赖 Processor 或 Exporter。
5. `Span.makeCurrent()` 把当前 Span 放进 `Context`，因此内部日志有机会关联 trace id 和 span id。
6. `span.end()`、metric reader collect、log emit 分别触发三类信号进入 SDK 后续处理。
7. `forceFlush()` 和 `shutdown()` 保证学习示例在退出前尽量把缓冲数据输出并释放资源。

如果把 logging exporter 换成 OTLP exporter，整体结构不变，只是 Exporter 从“本地输出”变成“发送到 Collector 或后端”。这也是 OpenTelemetry API、SDK 和 Exporter 分层的价值：业务埋点代码基本不需要跟着后端变化而改动。
