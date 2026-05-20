# 传播器

## 学习目标

- 理解 Propagator 的职责。
- 掌握 inject 和 extract 的流程。
- 理解 Trace Context、Baggage 和其他传播格式。

## 前置知识

- HTTP、RPC metadata 或消息队列 headers。
- W3C Trace Context。
- Context API。

## 核心概念

- Propagator 负责在进程边界注入和提取上下文。
- TextMapPropagator 面向字符串键值载体。
- 传播格式决定跨语言、跨服务互操作能力。

## 源码入口

- `api/trace/propagation/`
- `api/baggage/propagation/`
- `extensions/trace-propagators/`

## 关键类与接口

- `TextMapPropagator`
- `TextMapSetter`
- `TextMapGetter`
- `W3CTraceContextPropagator`
- `W3CBaggagePropagator`
- `ContextPropagators`

## 运行流程

- 客户端从当前 `Context` 读取上下文。
- 调用 propagator inject 写入 carrier。
- 服务端从 carrier 调用 extract。
- extract 得到的新 `Context` 成为服务端请求处理上下文。

## 细化知识点

- inject 和 extract 的职责边界。
- carrier、getter、setter。
- 多个 Propagator 的组合。
- W3C Trace Context header 字段。
- B3、Jaeger 等兼容格式。

## 常见问题

- 为什么服务间调用需要传播器？
- Propagator 和 Context 是什么关系？
- 多种传播格式同时启用会怎样？

## 实践任务

- 手写一个 Map carrier 的 inject/extract 示例。
- 阅读 W3C Trace Context 的解析流程。
- 配置组合 Propagator。

## 延伸阅读

- W3C Trace Context。
- W3C Baggage。

## 代码示例：Map carrier 中注入和提取上下文

```java
import io.opentelemetry.api.trace.Span;
import io.opentelemetry.api.trace.Tracer;
import io.opentelemetry.api.trace.propagation.W3CTraceContextPropagator;
import io.opentelemetry.context.Context;
import io.opentelemetry.context.propagation.TextMapGetter;
import io.opentelemetry.context.propagation.TextMapPropagator;
import io.opentelemetry.context.propagation.TextMapSetter;
import java.util.Map;

class PropagationExample {
  private static final TextMapSetter<Map<String, String>> SETTER = Map::put;

  private static final TextMapGetter<Map<String, String>> GETTER =
      new TextMapGetter<Map<String, String>>() {
        @Override
        public Iterable<String> keys(Map<String, String> carrier) {
          return carrier.keySet();
        }

        @Override
        public String get(Map<String, String> carrier, String key) {
          return carrier.get(key);
        }
      };

  void client(Tracer tracer, Map<String, String> headers) {
    Span span = tracer.spanBuilder("client").startSpan();
    try {
      Context context = Context.current().with(span);
      TextMapPropagator propagator = W3CTraceContextPropagator.getInstance();

      // inject 把当前 Context 编码到 headers，例如 traceparent。
      propagator.inject(context, headers, SETTER);
    } finally {
      span.end();
    }
  }

  Context server(Map<String, String> headers) {
    TextMapPropagator propagator = W3CTraceContextPropagator.getInstance();

    // extract 从 headers 中解析远端父上下文。
    return propagator.extract(Context.current(), headers, GETTER);
  }
}
```

传播器不负责发送 HTTP 请求，也不负责创建 Span。它只做一件事：把 `Context` 和跨进程载体互相转换。

## Header 观察点

W3C Trace Context 常见 header 是 `traceparent`。如果服务间 Trace 断链，可以先抓包或打印请求头，确认：

- 上游是否注入了 `traceparent`。
- 下游是否提取了 `traceparent`。
- 下游创建 server span 时是否使用了提取后的 Context。
