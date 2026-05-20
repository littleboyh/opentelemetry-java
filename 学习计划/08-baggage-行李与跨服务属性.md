# Baggage 与跨服务属性

## 学习目标

- 理解 Baggage 的用途和边界。
- 掌握 Baggage 如何随 Context 跨服务传播。
- 区分 Baggage、Span Attribute 和 Resource Attribute。

## 前置知识

- HTTP Header。
- 分布式上下文传播。
- 数据隐私与字段治理。

## 核心概念

- Baggage 是可跨进程传播的键值集合。
- Baggage 适合少量、明确需要跨服务传播的业务上下文。
- Baggage 不等同于 Span 属性，也不应滥用。

## 源码入口

- `api/baggage/`
- `api/incubator/`
- `context/`

## 关键类与接口

- `Baggage`
- `BaggageBuilder`
- `BaggageEntry`
- `BaggageContextKey`

## 运行流程

- 创建或读取当前 Baggage。
- 写入新的键值。
- 将 Baggage 放入 `Context`。
- 通过 Propagator 注入到请求载体。
- 下游从请求载体提取 Baggage。

## 细化知识点

- Baggage header 格式。
- metadata 的含义。
- Baggage 大小和数量限制。
- 隐私数据风险。
- Baggage 与 sampling 的潜在关系。

## 常见问题

- 什么信息适合放入 Baggage？
- Baggage 会自动变成 Span 属性吗？
- 为什么 Baggage 需要严格控制？

## 实践任务

- 写一个设置和读取 Baggage 的示例。
- 找到 Baggage 与 Context 的绑定代码。
- 记录 Baggage 与 Attributes 的差异。

## 延伸阅读

- W3C Baggage specification。

## 代码示例：设置和读取 Baggage

```java
import io.opentelemetry.api.baggage.Baggage;
import io.opentelemetry.context.Scope;

class BaggageExample {
  void handleRequest() {
    Baggage baggage =
        Baggage.builder()
            .put("tenant.id", "tenant-a")
            .put("experiment", "checkout-v2")
            .build();

    try (Scope ignored = baggage.makeCurrent()) {
      callDownstream();
    }
  }

  void callDownstream() {
    String tenantId = Baggage.current().getEntryValue("tenant.id");
    // tenantId 可以被用于少量跨服务决策，但不要放敏感信息。
  }
}
```

Baggage 会跨服务传播，所以它比普通 Span 属性更敏感。不要放密码、token、身份证号、完整用户信息等隐私数据；也不要放高基数字段。

## 与 Attribute 和 Resource 的区别

| 类型 | 生命周期 | 是否跨服务传播 | 典型内容 |
| --- | --- | --- | --- |
| Baggage | 当前上下文 | 是 | tenant、实验分组、路由提示 |
| Span Attribute | 单个 Span | 否，除非后端展示 | HTTP 方法、状态码、业务标签 |
| Resource Attribute | Provider/进程级 | 随遥测数据导出 | service.name、host、container |

源码阅读时可以从 `api/all/src/main/java/io/opentelemetry/api/baggage/Baggage.java` 开始，再看 W3C Baggage propagator 如何把它写入 carrier。
