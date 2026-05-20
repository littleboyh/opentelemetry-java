# 源码阅读路线

## 学习目标

- 建立可执行的源码阅读顺序。
- 避免一开始陷入实现细节。
- 把 API、SDK 和测试串起来阅读。

## 前置知识

- Java 包和模块导航。
- Gradle 子项目。
- IDE 跳转与调用层级。

## 核心概念

- 先读接口和测试，再读实现。
- 先读一条完整数据链路，再横向比较其他信号。
- 先理解稳定公共 API，再进入 internal 实现。

## 源码入口

- `api/`
- `context/`
- `sdk/trace/`
- `sdk/metrics/`
- `sdk/logs/`
- `exporters/`

## 关键类与接口

- 待随阅读进度补充。

## 运行流程

- 从最小 API 示例开始。
- 跳到 SDK Provider。
- 跟踪数据对象创建。
- 跟踪 Processor 或 Reader。
- 跟踪 Exporter。
- 回到测试验证理解。

## 细化知识点

- Trace 阅读路线。
- Metrics 阅读路线。
- Logs 阅读路线。
- Context 阅读路线。
- Autoconfigure 阅读路线。
- Exporter 阅读路线。

## 常见问题

- 读源码时应该从测试还是实现开始？
- 遇到 internal 包是否需要全部读完？
- 如何判断一个类是公共契约还是实现细节？

## 实践任务

- 为每章记录 5 个关键类。
- 每读完一个组件画一张时序图。
- 每章写一个最小可运行示例或测试。

## 延伸阅读

- `docs/knowledge/general-patterns.md`
- `docs/knowledge/build.md`

## 推荐阅读顺序：从一条 Trace 开始

```text
api/trace/Tracer
  -> sdk/trace/SdkTracer
  -> sdk/trace/SdkSpanBuilder
  -> sdk/trace/SdkSpan
  -> sdk/trace/SpanProcessor
  -> sdk/trace/export/BatchSpanProcessor
  -> sdk/trace/export/SpanExporter
```

先把 Trace 读通，再迁移到 Metrics 和 Logs。这样做的好处是：Trace 有最直观的生命周期，能帮助你理解 Provider、Processor、Exporter 的共同模式。

## 每读一个类时问的五个问题

1. 这个类属于 API、SDK、Exporter 还是扩展层？
2. 它是公共契约、稳定实现、internal 实现，还是测试工具？
3. 它由谁创建？
4. 它调用谁？
5. 它的数据什么时候被释放或导出？

## 建议记录模板

```markdown
### 类名

- 路径：
- 所属层级：
- 主要职责：
- 关键字段：
- 关键方法：
- 上游调用者：
- 下游依赖：
- 容易误解：
```

这个模板可以直接复制到每个章节中，用于补充源码笔记。

## 搜索命令

```bash
rg -n "class SdkTracerProvider|interface SpanProcessor" sdk/trace -g '*.java'
rg -n "makeCurrent\\(|Context.current\\(" context api sdk -g '*.java'
rg -n "CompletableResultCode" sdk exporters -g '*.java'
```

搜索时先限定模块目录，再扩大范围。这样能减少无关结果，也更容易看出模块边界。
