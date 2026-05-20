# OpenTelemetry Java 学习计划

这套文档的目标是把 OpenTelemetry Java 的核心组件整理成一本易读的书。阅读顺序从整体架构开始，再进入上下文、API、SDK、三类信号、导出链路、自动配置、测试与扩展。

## 阅读路线

1. [前言](00-preface-前言.md)
2. [全局概览](01-overview-全局概览.md)
3. [上下文传播](02-context-上下文传播.md)
4. [公共 API](03-api-公共API.md)
5. [SDK 架构](04-sdk-SDK架构.md)
6. [链路追踪](05-trace-链路追踪.md)
7. [指标](06-metrics-指标.md)
8. [日志](07-logs-日志.md)
9. [Baggage 与跨服务属性](08-baggage-行李与跨服务属性.md)
10. [传播器](09-propagators-传播器.md)
11. [资源模型](10-resources-资源模型.md)
12. [采样](11-samplers-采样.md)
13. [处理器](12-processors-处理器.md)
14. [导出器](13-exporters-导出器.md)
15. [自动配置](14-autoconfigure-自动配置.md)
16. [测试工具与验证](15-testing-测试工具与验证.md)
17. [扩展机制](16-extension-扩展机制.md)
18. [源码阅读路线](17-source-reading-源码阅读路线.md)
19. [实践项目](18-practice-实践项目.md)
20. [术语表](99-glossary-术语表.md)

## 建议写作方式

- 每章先回答“这个组件解决什么问题”。
- 再说明“用户如何使用”和“SDK 内部如何实现”。
- 每章都保留源码入口、关键接口、运行流程和实践任务。
- 不急于写成最终版，先把问题、源码位置和验证方法记录下来。

## 分阶段学习计划

### 第一阶段：建立全局地图

先读 [前言](00-preface-前言.md)、[全局概览](01-overview-全局概览.md)、[公共 API](03-api-公共API.md)。这一阶段不追求看懂所有实现细节，只需要回答三个问题：

- 业务代码为什么应该优先依赖 API？
- SDK 为什么是可替换、可配置的实现层？
- Context 为什么是三类信号共享的基础能力？

建议产出一张组件关系图：`Application -> API -> SDK -> Processor/Reader -> Exporter -> Backend`。

### 第二阶段：打通 Trace 主链路

依次阅读 [上下文传播](02-context-上下文传播.md)、[链路追踪](05-trace-链路追踪.md)、[采样](11-samplers-采样.md)、[处理器](12-processors-处理器.md)、[导出器](13-exporters-导出器.md)。Trace 是最适合入门源码的信号，因为它有清晰的生命周期：创建 Span、设为当前、记录属性、结束、处理、导出。

这一阶段要重点理解：

- `Span.current()` 为什么能拿到当前 Span。
- `Sampler` 在 Span 创建时做了什么决策。
- `BatchSpanProcessor` 为什么需要队列和后台线程。
- `SpanExporter` 为什么返回 `CompletableResultCode`。

### 第三阶段：横向比较 Metrics 与 Logs

阅读 [指标](06-metrics-指标.md) 和 [日志](07-logs-日志.md)。不要把 Metrics 和 Logs 当成 Trace 的复制品，它们的数据模型和导出节奏不同：

- Metrics 以聚合和采集周期为中心。
- Logs 以结构化事件和上下文关联为中心。
- Trace 以 Span 生命周期为中心。

### 第四阶段：补齐跨服务与自动化配置

阅读 [Baggage](08-baggage-行李与跨服务属性.md)、[传播器](09-propagators-传播器.md)、[资源模型](10-resources-资源模型.md)、[自动配置](14-autoconfigure-自动配置.md)、[扩展机制](16-extension-扩展机制.md)。这一阶段关注“多个服务、多种运行环境、多种后端”下 SDK 如何自动组装。

### 第五阶段：用测试和实践巩固

最后阅读 [测试工具与验证](15-testing-测试工具与验证.md)、[源码阅读路线](17-source-reading-源码阅读路线.md)、[实践项目](18-practice-实践项目.md)、[术语表](99-glossary-术语表.md)。每读完一章，至少做一个小实验，并记录：

- 使用了哪些 API。
- 触发了哪些 SDK 类。
- 产生了什么遥测数据。
- 如何验证结果。

## 统一章节结构

每个主题文件默认包含以下部分：

- 学习目标
- 前置知识
- 核心概念
- 源码入口
- 关键类与接口
- 运行流程
- 细化知识点
- 常见问题
- 实践任务
- 延伸阅读

## 最小实验环境建议

学习时可以先使用内存导出器或 logging exporter，不必一开始就接入完整后端。推荐顺序：

```java
// 1. 先用 API 写业务侧埋点代码。
// 2. 再用 SDK 接上 InMemory 或 Logging exporter。
// 3. 最后接入 OTLP exporter 和 OpenTelemetry Collector。
```

这个顺序能把“业务代码怎么写”和“数据怎么被导出”拆开，减少排查难度。
