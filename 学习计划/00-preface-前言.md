# 前言

## 学习目标

- 明确这套学习计划的边界：聚焦 OpenTelemetry Java 核心组件。
- 建立从使用、概念、源码到实践的阅读节奏。
- 形成可持续扩写成书的文档结构。

## 前置知识

- Java 基础与并发基础。
- Gradle 多模块项目的基本结构。
- 可观测性中的 Trace、Metric、Log 基本概念。

## 核心概念

- OpenTelemetry 是一套跨语言的可观测性标准。
- OpenTelemetry Java 包含 API、SDK、Exporter、扩展和测试工具等模块。
- API 面向业务代码，SDK 面向数据处理和导出。

## 源码入口

- `README.md`
- `api/`
- `context/`
- `sdk/`
- `exporters/`
- `extensions/`

## 关键类与接口

- `OpenTelemetry`：API 总入口。
- `GlobalOpenTelemetry`：全局 API 入口。
- `Context`：进程内上下文容器。
- `Tracer` / `Meter` / `Logger`：三类信号的 API 入口。
- `SdkTracerProvider` / `SdkMeterProvider` / `SdkLoggerProvider`：三类信号的 SDK Provider。

## 运行流程

- 从一个业务操作开始创建遥测数据。
- 通过 API 写入 Span、Metric 或 Log。
- 由 SDK 处理、采样、聚合、批量发送。
- 通过 Exporter 发送到后端系统。

## 细化知识点

- OpenTelemetry 规范与 Java 实现的关系。
- API 和 SDK 的职责边界。
- 稳定模块与 alpha 模块的区别。
- 手动埋点与自动埋点的关系。

## 常见问题

- 为什么业务代码通常只依赖 API？
- 为什么 SDK 配置通常放在应用启动阶段？
- Java Agent 和这个仓库是什么关系？

## 实践任务

- 阅读项目根目录 `README.md`。
- 画出 API、SDK、Exporter 的关系图。
- 记录每个顶层目录的职责。

## 延伸阅读

- OpenTelemetry 官方文档。
- OpenTelemetry Specification。

## 本书的阅读约定

本书使用“先用起来，再看源码”的方式组织内容。每章都尽量包含三层信息：

1. **使用层**：应用或库作者应该怎么调用 OpenTelemetry API。
2. **实现层**：SDK 内部大致如何接收、处理和导出数据。
3. **验证层**：如何用测试、内存导出器或本地 Collector 验证理解。

示例代码会尽量写成小片段，而不是完整应用。这样读者可以把片段复制到自己的实验项目中，再根据需要补充依赖和启动代码。

## 建议记录方式

每读一个组件，建议在章节里补充一张表：

| 问题 | 记录 |
| --- | --- |
| 这个组件解决什么问题？ | 用一句话描述组件的职责。 |
| 用户直接接触哪些 API？ | 记录 public API 和常用 builder。 |
| SDK 内部哪些类最关键？ | 记录 Provider、Processor、Reader、Exporter 等实现类。 |
| 和其他组件如何协作？ | 画出上游调用者和下游依赖。 |
| 有什么容易误解的地方？ | 记录边界、生命周期和线程相关问题。 |

这张表能帮助学习从“读过源码”变成“能讲清楚设计”。
