# 上下文传播

## 学习目标

- 理解 `Context` 在 OpenTelemetry Java 中的核心地位。
- 掌握上下文如何跨方法、线程和异步边界传播。
- 区分 `Context`、`Scope`、当前 Span 的关系。

## 前置知识

- Java `ThreadLocal`。
- try-with-resources。
- 异步编程和线程池。

## 核心概念

- `Context` 是不可变的键值容器。
- `Scope` 表示把某个 `Context` 设为当前上下文的一段生命周期。
- 当前 Span 通常通过 `Context` 读取，而不是通过全局变量读取。

## 源码入口

- `context/`
- `api/all/`
- `api/trace/`

## 关键类与接口

- `Context`
- `ContextKey`
- `Scope`
- `ContextStorage`
- `ImplicitContextKeyed`

## 运行流程

- 从当前线程读取当前 `Context`。
- 向 `Context` 写入 Span 或其他值。
- 调用 `makeCurrent()` 创建 `Scope`。
- 在 `Scope` 关闭时恢复之前的上下文。

## 细化知识点

- 上下文不可变设计。
- `Context.current()` 的读取路径。
- `Scope` 泄漏的风险。
- 跨线程传播的手动包装方式。
- Context Storage 的扩展点。

## 常见问题

- 为什么不能只用 `ThreadLocal<Span>`？
- 忘记关闭 `Scope` 会发生什么？
- 异步任务里为什么经常丢失当前 Span？

## 实践任务

- 写一个嵌套 `Scope` 的小例子。
- 阅读 `Context` 的核心方法。
- 追踪 `Span.current()` 如何从 `Context` 取值。

## 延伸阅读

- OpenTelemetry Context specification。

## 代码示例：自定义 Context 值

```java
import io.opentelemetry.context.Context;
import io.opentelemetry.context.ContextKey;
import io.opentelemetry.context.Scope;

class ContextExample {
  private static final ContextKey<String> USER_ID = ContextKey.named("user-id");

  void handleRequest() {
    Context context = Context.current().with(USER_ID, "alice");

    // makeCurrent() 会把 context 设为当前线程上的当前上下文。
    // Scope.close() 会恢复之前的上下文，所以一定要用 try-with-resources。
    try (Scope ignored = context.makeCurrent()) {
      readCurrentUser();
    }
  }

  void readCurrentUser() {
    String userId = Context.current().get(USER_ID);
    // userId == "alice"
  }
}
```

源码阅读时重点看 `context/src/main/java/io/opentelemetry/context/Context.java` 的 `with()` 和 `makeCurrent()`。`Context` 本身是不可变的，`with()` 返回新对象；当前上下文的保存和恢复由 `ContextStorage` 完成。

## 代码示例：跨线程传播

```java
import io.opentelemetry.context.Context;
import java.util.concurrent.Executor;

class AsyncContextExample {
  void submit(Executor executor) {
    Context captured = Context.current();

    executor.execute(
        () -> {
          // 新线程默认拿不到提交线程的 ThreadLocal。
          // wrap() 会在任务执行时恢复 captured context。
          captured.wrap(this::runInWorker).run();
        });
  }

  private void runInWorker() {}
}
```

异步场景最常见的问题是“Span 断链”。排查时先确认任务提交处是否捕获了 `Context.current()`，再确认任务执行处是否恢复了这个 Context。
