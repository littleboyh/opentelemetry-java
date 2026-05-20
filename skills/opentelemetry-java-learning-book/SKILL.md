---
name: opentelemetry-java-learning-book
description: Use when working in the opentelemetry-java repository and the user asks to scaffold, organize, or expand a Chinese study plan, learning directory, component-by-component reading guide, or book-style Markdown documentation for OpenTelemetry Java core components.
---

# OpenTelemetry Java Learning Book

## Overview

Create a book-like Markdown learning plan for OpenTelemetry Java. Favor a readable component-by-component structure that helps the user learn API, SDK, context propagation, signals, exporters, autoconfigure, testing, and extension points.

## Workflow

1. Read repository guidance first:
   - Start with `AGENTS.md` if present.
   - Read `docs/knowledge/README.md`.
   - For documentation-only scaffolding, load `docs/knowledge/general-patterns.md` only if style or naming questions arise.

2. Locate the learning directory:
   - Prefer a user-provided path.
   - If the user says they already created one, search shallowly with `find . -maxdepth 3 -type d` for names like `学习计划`, `learn`, `study`, or `plan`.
   - Do not rename the directory without explicit request.

3. Confirm naming style if unspecified:
   - Recommend `01-overview-全局概览.md` style: numbered English slug plus Chinese title.
   - Offer alternatives only when the user has not already chosen a style.

4. Propose the structure before writing files:
   - Recommend organizing by core components rather than by learning phase or signal type.
   - Explain the tradeoff briefly: component chapters map well to source modules and later work as a reference book.
   - Wait for user approval before creating files.

5. Create the root README and chapter files:
   - Root `README.md` is the table of contents and reading route.
   - Each chapter gets a consistent skeleton:
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

6. Verify:
   - List created Markdown files.
   - Check README links point to actual files.
   - Report that no Gradle tests were run if only documentation was changed.

## Recommended Chapter Set

Use this list unless the user asks for a smaller or different scope:

```text
README.md
00-preface-前言.md
01-overview-全局概览.md
02-context-上下文传播.md
03-api-公共API.md
04-sdk-SDK架构.md
05-trace-链路追踪.md
06-metrics-指标.md
07-logs-日志.md
08-baggage-行李与跨服务属性.md
09-propagators-传播器.md
10-resources-资源模型.md
11-samplers-采样.md
12-processors-处理器.md
13-exporters-导出器.md
14-autoconfigure-自动配置.md
15-testing-测试工具与验证.md
16-extension-扩展机制.md
17-source-reading-源码阅读路线.md
18-practice-实践项目.md
99-glossary-术语表.md
```

## Chapter Guidance

Use concise, practical content. Each chapter should help the reader answer:

- 这个组件解决什么问题？
- 用户代码如何使用它？
- SDK 内部如何实现或处理它？
- 应该从哪些源码目录和关键类开始读？
- 可以做什么小实验验证理解？

## Source Entry Hints

- Overview: `api/`, `context/`, `sdk/`, `exporters/`, `extensions/`, `sdk-extensions/`
- Context: `context/`, `api/trace/`
- API: `api/`, `api/all/`, `api/incubator/`
- SDK: `sdk/common/`, `sdk/trace/`, `sdk/metrics/`, `sdk/logs/`
- Trace: `api/trace/`, `sdk/trace/`, `exporters/otlp/`
- Metrics: `api/metrics/`, `sdk/metrics/`, `exporters/prometheus/`
- Logs: `api/logs/`, `sdk/logs/`
- Propagators: `api/trace/propagation/`, `api/baggage/propagation/`, `extensions/trace-propagators/`
- Autoconfigure: `sdk-extensions/autoconfigure/`, `sdk-extensions/autoconfigure-spi/`
- Testing: `testing-internal/`, `sdk/testing/`, `integration-tests/`

## Verification Commands

Use commands like these after creating or updating the book scaffold:

```bash
find 学习计划 -maxdepth 1 -type f -name '*.md' | sort
rg -n "\[.*\]\(.*\.md\)" 学习计划/README.md
while IFS= read -r link; do test -f "学习计划/$link" || { echo "missing: $link"; exit 1; }; done < <(rg -o '\([^)]*\.md\)' 学习计划/README.md | sed 's/[()]//g')
```

Adjust `学习计划` to the actual learning directory.
