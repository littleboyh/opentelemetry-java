# 公共 API

## 学习目标

- 理解 OpenTelemetry Java API 的设计目标。
- 掌握业务代码应该依赖哪些接口。
- 理解 API 稳定性和兼容性要求。

## 前置知识

- Java interface 与默认实现。
- 语义化版本与 API 兼容。
- No-op 实现模式。

## 核心概念

- API 是埋点代码和 SDK 实现之间的稳定契约。
- 没有 SDK 时，API 仍应以 no-op 方式安全运行。
- API 模块尽量避免引入具体实现细节。

## 源码入口

- `api/`
- `api/all/`
- `api/incubator/`

## 关键类与接口

- `OpenTelemetry`
- `GlobalOpenTelemetry`
- `Tracer`
- `Meter`
- `Logger`
- `Attributes`
- `AttributeKey`

## 运行流程

- 业务代码获取 `OpenTelemetry`。
- 从中获取 `Tracer`、`Meter` 或 `Logger`。
- 创建 Span、Instrument 或 LogRecord。
- API 把调用转交给 SDK 或 no-op 实现。

## 细化知识点

- Global API 的初始化规则。
- `Attributes` 的不可变性。
- Instrumentation scope 的命名。
- API 包的稳定性边界。
- incubator API 的使用风险。

## 常见问题

- 业务库应该依赖 API 还是 SDK？
- 什么时候可以使用 `GlobalOpenTelemetry`？
- 为什么 API 里有 no-op 实现？

## 实践任务

- 找到 `OpenTelemetry.noop()` 的实现路径。
- 用 API 写一个最小手动 Span 示例。
- 比较 API 模块和 SDK 模块的依赖差异。

## 延伸阅读

- `docs/knowledge/api-design.md`
- `VERSIONING.md`

## 代码示例：库代码只依赖 API

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;

public final class PaymentClient {
  private final Tracer tracer;

  public PaymentClient(OpenTelemetry openTelemetry) {
    this.tracer = openTelemetry.getTracer("com.example.payment-client");
  }

  public void pay(String orderId) {
    Span span = tracer.spanBuilder("PaymentClient.pay").startSpan();
    try {
      span.setAttribute("order.id", orderId);
      // 调用外部支付系统。
    } finally {
      span.end();
    }
  }
}
```

库代码不应该自己创建 `SdkTracerProvider`。库只负责“产生遥测数据”，应用负责决定“如何采样、处理和导出”。这就是 API 与 SDK 分离的核心价值。

## No-op 的意义

如果应用没有安装 SDK，API 调用仍然应该安全运行：

```java
OpenTelemetry openTelemetry = OpenTelemetry.noop();
Tracer tracer = openTelemetry.getTracer("demo");

// 这里可以正常调用，但不会产生真实导出数据。
Span span = tracer.spanBuilder("noop-span").startSpan();
span.end();
```

阅读 `api/all/src/main/java/io/opentelemetry/api/DefaultOpenTelemetry.java` 和各类 `Default*` 实现时，要注意它们如何保证“无 SDK 时也不影响业务代码”。
