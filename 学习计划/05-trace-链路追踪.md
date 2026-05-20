# 链路追踪

## 学习目标

- 理解 Trace、Span、SpanContext 的关系。
- 掌握 Span 创建、属性、事件、状态和结束流程。
- 理解采样、处理器和导出器如何参与 Trace 管线。

## 前置知识

- 分布式调用链。
- W3C Trace Context。
- 同步和异步调用。

## 核心概念

- Trace 是一次请求在分布式系统中的完整路径。
- Span 表示路径中的一个操作。
- SpanContext 携带 trace id、span id、trace flags 等跨进程传播信息。

## 源码入口

- `api/trace/`
- `sdk/trace/`
- `exporters/otlp/`

## 关键类与接口

- `Tracer`
- `Span`
- `SpanBuilder`
- `SpanContext`
- `SdkTracerProvider`
- `SpanProcessor`
- `SpanExporter`

## 运行流程

- 获取 `Tracer`。
- 使用 `spanBuilder()` 创建 Span。
- 将 Span 放入当前 `Context`。
- 设置属性、事件和状态。
- 结束 Span。
- SDK 通过 Processor 和 Exporter 处理 Span。

## 细化知识点

- root span 与 child span。
- SpanKind 的语义。
- Span 属性、事件、链接和状态。
- Span limits。
- ended span 的不可变性。
- BatchSpanProcessor 的批处理机制。

## 常见问题

- 为什么 Span 一定要 end？
- `Span.current()` 和手动传 Span 有什么区别？
- link 和 parent-child 有什么区别？

## 实践任务

- 写一个父子 Span 示例。
- 找出 Span 被采样和未采样时的行为差异。
- 阅读 BatchSpanProcessor 的队列和导出逻辑。

## 延伸阅读

- OpenTelemetry Trace specification。
- W3C Trace Context。

## Span 生命周期示例

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.StatusCode;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.context.Scope;

class TraceExample {
  private final Tracer tracer;

  TraceExample(Tracer tracer) {
    this.tracer = tracer;
  }

  void handleHttpRequest() {
    Span serverSpan = tracer.spanBuilder("GET /checkout").startSpan();

    try (Scope ignored = serverSpan.makeCurrent()) {
      serverSpan.setAttribute("http.request.method", "GET");
      serverSpan.setAttribute("url.path", "/checkout");

      callPaymentService();
      serverSpan.setStatus(StatusCode.OK);
    } catch (RuntimeException e) {
      serverSpan.recordException(e);
      serverSpan.setStatus(StatusCode.ERROR, e.getMessage());
      throw e;
    } finally {
      serverSpan.end();
    }
  }

  private void callPaymentService() {
    // 如果当前 Context 中已有 serverSpan，这个 Span 默认会成为它的子 Span。
    Span clientSpan = tracer.spanBuilder("POST payment").startSpan();
    try (Scope ignored = clientSpan.makeCurrent()) {
      clientSpan.setAttribute("server.address", "payment.internal");
    } finally {
      clientSpan.end();
    }
  }
}
```

学习 Trace 时要始终围绕一个问题：Span 在哪里开始，在哪里成为当前 Span，在哪里结束，结束后交给谁。

## 源码阅读主线

1. 从 `api/all/src/main/java/io/opentelemetry/api/trace/Tracer.java` 看用户如何创建 Span。
2. 跳到 `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/SdkTracer.java` 看 SDK 如何实现。
3. 进入 `SdkSpan` 看属性、事件、状态和 `end()` 行为。
4. 进入 `SpanProcessor`、`BatchSpanProcessor` 看结束后的处理。
5. 进入 `SpanExporter` 看导出边界。

## 常见误区

- `Span.current()` 不是全局单例读取，而是从当前 `Context` 读取。
- `span.makeCurrent()` 不会自动 `span.end()`，二者生命周期不同。
- `recordException()` 不等于自动把 Span 状态设为 ERROR，实际代码中通常两个都做。
