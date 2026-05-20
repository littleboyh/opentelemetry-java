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
