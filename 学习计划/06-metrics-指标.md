# 指标

## 学习目标

- 理解 OpenTelemetry Metrics 的数据模型。
- 掌握 Instrument、Reader、View、Aggregation 的关系。
- 理解同步和异步指标的差异。

## 前置知识

- Counter、Gauge、Histogram 等指标类型。
- 时间序列数据。
- 聚合窗口和 temporality。

## 核心概念

- Instrument 是用户记录指标的 API。
- SDK 聚合测量值并通过 MetricReader 导出。
- View 可以改变聚合、名称、描述和属性过滤。

## 源码入口

- `api/metrics/`
- `sdk/metrics/`
- `exporters/otlp/`
- `exporters/prometheus/`

## 关键类与接口

- `Meter`
- `LongCounter`
- `DoubleHistogram`
- `ObservableLongGauge`
- `SdkMeterProvider`
- `MetricReader`
- `MetricExporter`
- `View`

## 运行流程

- 获取 `Meter`。
- 创建同步或异步 Instrument。
- 记录测量值。
- SDK 根据 View 和默认规则聚合。
- Reader 收集数据并交给 Exporter。

## 细化知识点

- 同步 Instrument 与异步 Instrument。
- cumulative 和 delta temporality。
- histogram bucket。
- attribute cardinality。
- exemplar。
- Prometheus pull 模式与 OTLP push 模式。

## 常见问题

- Counter 和 UpDownCounter 有什么区别？
- Gauge 在 OpenTelemetry 中为什么通常是异步的？
- View 应该解决什么问题？

## 实践任务

- 写一个 Counter 和 Histogram 示例。
- 配置一个 View 修改 Histogram bucket。
- 阅读 PeriodicMetricReader 的采集流程。

## 延伸阅读

- OpenTelemetry Metrics specification。

## 代码示例：Counter 与 Histogram

```java
import io.opentelemetry.api.metrics.DoubleHistogram;
import io.opentelemetry.api.metrics.LongCounter;
import io.opentelemetry.api.metrics.Meter;
import io.opentelemetry.api.common.Attributes;
import static io.opentelemetry.api.common.AttributeKey.stringKey;

class MetricsExample {
  private final LongCounter requestCounter;
  private final DoubleHistogram latencyHistogram;

  MetricsExample(Meter meter) {
    requestCounter =
        meter
            .counterBuilder("checkout.requests")
            .setDescription("Number of checkout requests")
            .setUnit("{request}")
            .build();

    latencyHistogram =
        meter
            .histogramBuilder("checkout.latency")
            .setDescription("Checkout latency")
            .setUnit("ms")
            .build();
  }

  void record(long latencyMillis, String status) {
    Attributes attributes = Attributes.of(stringKey("status"), status);

    // Counter 只能递增，适合记录次数、字节数等单调增长值。
    requestCounter.add(1, attributes);

    // Histogram 记录分布，适合延迟、请求大小等。
    latencyHistogram.record(latencyMillis, attributes);
  }
}
```

Metrics 的重点不是“每次调用都导出一条数据”，而是“SDK 如何按时间窗口和属性维度聚合测量值”。

## SDK 管线

```text
Meter -> Instrument -> Measurement
  -> Storage/Aggregation
  -> MetricReader.collect()
  -> MetricExporter.export()
```

阅读顺序建议：

1. `api/all/src/main/java/io/opentelemetry/api/metrics/Meter.java`
2. `sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/SdkMeterProvider.java`
3. `sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/export/PeriodicMetricReader.java`
4. `sdk/metrics/src/main/java/io/opentelemetry/sdk/metrics/export/MetricExporter.java`

## 属性基数提醒

指标属性会形成时间序列维度。不要把用户 ID、订单 ID、请求 ID 这类高基数字段直接放进 Metrics 属性，否则后端存储和查询成本会迅速上升。
