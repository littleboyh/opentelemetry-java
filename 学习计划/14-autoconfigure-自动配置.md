# 自动配置

## 学习目标

- 理解自动配置解决的问题。
- 掌握环境变量、系统属性和 SPI 的配置路径。
- 理解自动配置与手动构建 SDK 的关系。

## 前置知识

- Java `ServiceLoader`。
- 环境变量和系统属性。
- Builder 模式。

## 核心概念

- 自动配置把常见 SDK 初始化逻辑参数化。
- 配置来源通常包括环境变量、系统属性和 SPI。
- 自动配置适合应用启动阶段集中初始化。

## 源码入口

- `sdk-extensions/autoconfigure/`
- `sdk-extensions/autoconfigure-spi/`

## 关键类与接口

- `AutoConfiguredOpenTelemetrySdk`
- `ConfigProperties`
- `AutoConfigurationCustomizer`
- `AutoConfigurationCustomizerProvider`

## 运行流程

- 读取配置属性。
- 发现 SPI 扩展。
- 构建 Resource、Propagator、Sampler、Reader、Processor、Exporter。
- 组装 `OpenTelemetrySdk`。
- 可选注册为全局实例。

## 细化知识点

- 配置优先级。
- `otel.*` 属性命名。
- exporter 自动选择。
- ResourceProvider 自动发现。
- Customizer 的扩展点。
- 与 Java Agent 配置的关系。

## 常见问题

- 自动配置和手动配置能否混用？
- 属性配置不生效时如何排查？
- SPI 扩展加载顺序重要吗？

## 实践任务

- 用系统属性配置 OTLP exporter。
- 写一个最小 AutoConfigurationCustomizerProvider。
- 阅读自动配置的构建顺序。

## 延伸阅读

- OpenTelemetry Java autoconfigure 文档。

## 代码示例：自动配置入口

```java
import io.opentelemetry.api.OpenTelemetry;
import io.opentelemetry.sdk.autoconfigure.AutoConfiguredOpenTelemetrySdk;

class AutoconfigureExample {
  OpenTelemetry init() {
    return AutoConfiguredOpenTelemetrySdk.builder()
        // 注册为全局实例后，业务代码可以通过 GlobalOpenTelemetry.get() 获取。
        .setResultAsGlobal()
        .build()
        .getOpenTelemetrySdk();
  }
}
```

自动配置通常由系统属性或环境变量驱动，例如：

```bash
OTEL_SERVICE_NAME=checkout-service \
OTEL_TRACES_EXPORTER=otlp \
OTEL_EXPORTER_OTLP_ENDPOINT=http://localhost:4317 \
java -jar app.jar
```

## 自动配置构建顺序

```text
ConfigProperties
  -> Resource
  -> Propagators
  -> Sampler
  -> SpanProcessor / MetricReader / LogRecordProcessor
  -> Exporters
  -> OpenTelemetrySdk
```

阅读 `AutoConfiguredOpenTelemetrySdkBuilder` 时，要把它当成“SDK 装配器”看，而不是单个信号的实现类。

## SPI 扩展示例

```java
import io.opentelemetry.sdk.autoconfigure.spi.AutoConfigurationCustomizer;
import io.opentelemetry.sdk.autoconfigure.spi.AutoConfigurationCustomizerProvider;

public final class DemoCustomizerProvider implements AutoConfigurationCustomizerProvider {
  @Override
  public void customize(AutoConfigurationCustomizer customizer) {
    customizer.addPropertiesSupplier(() -> java.util.Collections.singletonMap(
        "otel.service.name", "checkout-service"));
  }
}
```

这个类需要通过 `META-INF/services/io.opentelemetry.sdk.autoconfigure.spi.AutoConfigurationCustomizerProvider` 声明，自动配置阶段才会被 `ServiceLoader` 发现。
