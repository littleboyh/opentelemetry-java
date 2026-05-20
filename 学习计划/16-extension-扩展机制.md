# 扩展机制

## 学习目标

- 理解 OpenTelemetry Java 的扩展边界。
- 掌握 SPI、Provider、自定义组件的常见入口。
- 区分稳定扩展点和内部实现细节。

## 前置知识

- Java SPI。
- Gradle 模块依赖。
- API 稳定性。

## 核心概念

- 扩展机制允许用户替换或补充 SDK 组件。
- 常见扩展包括 exporter、propagator、resource provider 和 autoconfigure customizer。
- internal 包通常不应作为扩展入口。

## 源码入口

- `extensions/`
- `sdk-extensions/`
- `api/incubator/`

## 关键类与接口

- `AutoConfigurationCustomizerProvider`
- `ConfigurableSpanExporterProvider`
- `ConfigurableMetricExporterProvider`
- `ConfigurableLogRecordExporterProvider`
- `ResourceProvider`

## 运行流程

- 实现指定 SPI 接口。
- 在 `META-INF/services` 中声明实现类。
- 自动配置阶段通过 ServiceLoader 加载。
- 扩展参与 SDK 构建。

## 细化知识点

- SPI 文件格式。
- exporter provider。
- propagator provider。
- resource provider。
- autoconfigure customizer。
- alpha 扩展点的兼容性风险。

## 常见问题

- 什么可以扩展，什么不应该扩展？
- 自定义 Exporter 应该实现哪个接口？
- 扩展点升级时如何处理兼容性？

## 实践任务

- 写一个自定义 ResourceProvider。
- 写一个 logging customizer。
- 找出项目中已有 SPI 声明。

## 延伸阅读

- `docs/knowledge/api-design.md`
- Java `ServiceLoader` 文档。

## 代码示例：ResourceProvider SPI

```java
import io.opentelemetry.api.common.Attributes;
import io.opentelemetry.sdk.autoconfigure.spi.ConfigProperties;
import io.opentelemetry.sdk.autoconfigure.spi.ResourceProvider;
import io.opentelemetry.sdk.resources.Resource;
import static io.opentelemetry.api.common.AttributeKey.stringKey;

public final class DemoResourceProvider implements ResourceProvider {
  @Override
  public Resource createResource(ConfigProperties config) {
    return Resource.create(
        Attributes.of(stringKey("deployment.region"), "local-dev"));
  }
}
```

服务声明文件：

```text
META-INF/services/io.opentelemetry.sdk.autoconfigure.spi.ResourceProvider
```

文件内容：

```text
com.example.DemoResourceProvider
```

自动配置启动时会通过 `ServiceLoader` 发现这个 Provider，并把它产出的 Resource 合并到 SDK Resource 中。

## 扩展边界

扩展时优先寻找公开 SPI 或 public API。不要依赖 `internal` 包里的类，因为这些类没有兼容性保证。`impl` 包虽然不是给应用开发者直接使用的 API，但在本项目中有更强的兼容性约束；具体边界要结合源码注释和版本策略判断。
