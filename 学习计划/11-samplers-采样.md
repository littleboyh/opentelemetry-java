# 采样

## 学习目标

- 理解采样在 Trace 管线中的位置。
- 掌握 head sampling 的决策过程。
- 区分采样决策、TraceFlags 和 Span 是否记录。

## 前置知识

- 分布式追踪成本控制。
- Trace ID 与 Span ID。
- 概率采样。

## 核心概念

- Sampler 在 Span 创建时决定是否采样。
- 采样结果影响 Span 是否记录和是否导出。
- 父 Span 的采样状态通常会影响子 Span。

## 源码入口

- `sdk/trace/`
- `api/trace/`

## 关键类与接口

- `Sampler`
- `SamplingResult`
- `SamplingDecision`
- `TraceIdRatioBasedSampler`
- `ParentBasedSampler`

## 运行流程

- 创建 Span 时调用 Sampler。
- Sampler 根据 parent context、trace id、name、kind、attributes、links 决策。
- SDK 创建 recording 或 non-recording Span。
- 采样 Span 进入 Processor 和 Exporter。

## 细化知识点

- always on、always off、ratio based。
- parent based sampling。
- remote parent 和 local parent。
- sampled flag 的传播。
- head sampling 与 tail sampling 的区别。

## 常见问题

- 未采样 Span 是否还会传播上下文？
- 采样率为什么不能简单等同于请求比例？
- tail sampling 为什么通常不在 SDK 内完成？

## 实践任务

- 配置不同 Sampler 观察 Span 输出差异。
- 阅读 ParentBasedSampler 的分支逻辑。
- 记录 sampled flag 在 HTTP header 中的变化。

## 延伸阅读

- OpenTelemetry Sampling specification。

## 代码示例：配置采样器

```java
import io.opentelemetry.sdk.trace.SdkTracerProvider;
import io.opentelemetry.sdk.trace.samplers.Sampler;

class SamplerExample {
  SdkTracerProvider tracerProvider() {
    return SdkTracerProvider.builder()
        // 父 Span 已采样时继续采样；没有父 Span 时按 Trace ID 概率采样。
        .setSampler(Sampler.parentBased(Sampler.traceIdRatioBased(0.10)))
        .build();
  }
}
```

`traceIdRatioBased(0.10)` 表示基于 Trace ID 做近似 10% 的 head sampling。它不是“每 10 个请求取 1 个”的计数采样，而是根据 Trace ID 稳定决策。

## 采样发生的位置

```text
tracer.spanBuilder(...)
  -> startSpan()
  -> Sampler.shouldSample(...)
  -> SamplingResult
  -> recording span 或 non-recording span
```

采样发生在 Span 创建时，所以 SDK 中常说的是 head sampling。此时还不知道整个 Trace 的完整形态，因此如果需要根据完整链路决定是否保留，通常要在 Collector 或后端做 tail sampling。

## 源码阅读重点

- `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/samplers/Sampler.java`
- `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/samplers/ParentBasedSampler.java`
- `sdk/trace/src/main/java/io/opentelemetry/sdk/trace/samplers/TraceIdRatioBasedSampler.java`

阅读时注意 `SamplingDecision` 的三个结果：丢弃、只传播、记录并采样。即使 Span 未采样，Trace Context 仍可能继续传播。
