# 资源模型

## 学习目标

- 理解 Resource 在遥测数据中的作用。
- 掌握 Resource Attribute 与 Span Attribute 的区别。
- 理解资源探测器和默认资源合并规则。

## 前置知识

- 服务元数据。
- 云平台、容器、主机信息。
- 语义约定。

## 核心概念

- Resource 描述产生遥测数据的实体。
- Resource Attribute 通常表示 service、host、process、container 等维度。
- SDK Provider 通常持有一个 Resource。

## 源码入口

- `sdk/common/`
- `sdk-extensions/`
- `semconv/` 如果当前版本包含相关模块。

## 关键类与接口

- `Resource`
- `ResourceBuilder`
- `ResourceProvider`
- `Attributes`

## 运行流程

- 创建默认 Resource。
- 通过配置或探测器补充属性。
- 合并多个 Resource。
- Provider 创建遥测数据时附带 Resource。
- Exporter 将 Resource 与数据一起发送。

## 细化知识点

- `service.name` 的重要性。
- Resource merge 规则。
- 默认 Resource。
- ResourceProvider SPI。
- Resource 与 Instrumentation Scope 的区别。

## 常见问题

- Resource 属性和 Span 属性应该如何选择？
- 为什么 `service.name` 经常缺失或不正确？
- 多个 Resource 合并时冲突怎么处理？

## 实践任务

- 创建自定义 Resource。
- 找出默认 Resource 的构建逻辑。
- 比较 OTLP 导出数据中的 Resource 和 Span 属性。

## 延伸阅读

- OpenTelemetry Resource specification。
- OpenTelemetry Semantic Conventions。

## 代码示例：自定义 Resource

```java
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.sdk.resources.Resource;
import static io.opentelemetry.api.common.AttributeKey.stringKey;

class ResourceExample {
  Resource resource() {
    Resource serviceResource =
        Resource.create(
            Attributes.of(
                stringKey("service.name"), "checkout-service",
                stringKey("service.version"), "1.2.3",
                stringKey("deployment.environment.name"), "dev"));

    // Resource.merge() 用于把默认资源、环境资源和手动资源合并。
    return Resource.getDefault().merge(serviceResource);
  }
}
```

Resource 的粒度通常是“产生遥测数据的实体”，例如一个服务进程、容器或主机。它不应该记录单次请求的动态信息。

## 判断属性应该放在哪里

```text
是否描述整个服务或进程？
  是 -> Resource Attribute
  否 -> 是否描述单个 Span / Log / Metric 点？
        是 -> Signal Attribute
        否 -> 是否必须跨服务传播？
              是 -> Baggage
              否 -> 重新评估是否需要记录
```

源码阅读重点：

- `sdk/common/src/main/java/io/opentelemetry/sdk/resources/Resource.java`
- `sdk/common/src/main/java/io/opentelemetry/sdk/resources/ResourceBuilder.java`
- `sdk-extensions/autoconfigure-spi/src/main/java/io/opentelemetry/sdk/autoconfigure/spi/ResourceProvider.java`
