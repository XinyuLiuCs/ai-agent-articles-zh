# 使用 Claude 构建 LLM 驱动的应用

*Building LLM-Powered Applications with Claude*---
https://github.com/anthropics/claude-code/tree/main/.skills/claude-api

发布日期：2025年

Published 2025

使用 Claude API 或 Anthropic SDK 构建应用。当代码导入 `anthropic`/`@anthropic-ai/sdk`/`claude_agent_sdk`，或用户要求使用 Claude API、Anthropic SDK 或 Agent SDK 时触发。当代码导入 `openai`/其他 AI SDK、通用编程或机器学习/数据科学任务时不触发。

Build apps with the Claude API or Anthropic SDK. TRIGGER when: code imports `anthropic`/`@anthropic-ai/sdk`/`claude_agent_sdk`, or user asks to use Claude API, Anthropic SDKs, or Agent SDK. DO NOT TRIGGER when: code imports `openai`/other AI SDK, general programming, or ML/data-science tasks.

# 使用 Claude 构建 LLM 驱动的应用

Building LLM-Powered Applications with Claude

本技能帮助你使用 Claude 构建 LLM 驱动的应用。根据你的需求选择合适的接入层，检测项目语言，然后阅读相关的语言专属文档。

This skill helps you build LLM-powered applications with Claude. Choose the right surface based on your needs, detect the project language, then read the relevant language-specific documentation.

## 默认设置

Defaults

除非用户另有要求：

Unless the user requests otherwise:

对于 Claude 模型版本，请使用 Claude Opus 4.6，可通过确切的模型字符串 `claude-opus-4-6` 访问。对于任何稍有复杂度的任务，请默认使用自适应思考（`thinking: {type: "adaptive"}`）。最后，对于任何可能涉及长输入、长输出或高 `max_tokens` 的请求，请默认使用流式传输——这可以防止请求超时。如果你不需要处理单个流事件，可以使用 SDK 的 `.get_final_message()` / `.finalMessage()` 辅助方法来获取完整响应。

For the Claude model version, please use Claude Opus 4.6, which you can access via the exact model string `claude-opus-4-6`. Please default to using adaptive thinking (`thinking: {type: "adaptive"}`) for anything remotely complicated. And finally, please default to streaming for any request that may involve long input, long output, or high `max_tokens` — it prevents hitting request timeouts. Use the SDK's `.get_final_message()` / `.finalMessage()` helper to get the complete response if you don't need to handle individual stream events

---

## 语言检测

Language Detection

在阅读代码示例之前，确定用户正在使用的编程语言：

Before reading code examples, determine which language the user is working in:

1. **查看项目文件**以推断语言：

1. **Look at project files** to infer the language:

   - `*.py`、`requirements.txt`、`pyproject.toml`、`setup.py`、`Pipfile` → **Python** ——从 `python/` 读取
   - `*.ts`、`*.tsx`、`package.json`、`tsconfig.json` → **TypeScript** ——从 `typescript/` 读取
   - `*.js`、`*.jsx`（不存在 `.ts` 文件）→ **TypeScript** —— JS 使用相同的 SDK，从 `typescript/` 读取
   - `*.java`、`pom.xml`、`build.gradle` → **Java** ——从 `java/` 读取
   - `*.kt`、`*.kts`、`build.gradle.kts` → **Java** —— Kotlin 使用 Java SDK，从 `java/` 读取
   - `*.scala`、`build.sbt` → **Java** —— Scala 使用 Java SDK，从 `java/` 读取
   - `*.go`、`go.mod` → **Go** ——从 `go/` 读取
   - `*.rb`、`Gemfile` → **Ruby** ——从 `ruby/` 读取
   - `*.cs`、`*.csproj` → **C#** ——从 `csharp/` 读取
   - `*.php`、`composer.json` → **PHP** ——从 `php/` 读取

   - `*.py`, `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile` → **Python** — read from `python/`
   - `*.ts`, `*.tsx`, `package.json`, `tsconfig.json` → **TypeScript** — read from `typescript/`
   - `*.js`, `*.jsx` (no `.ts` files present) → **TypeScript** — JS uses the same SDK, read from `typescript/`
   - `*.java`, `pom.xml`, `build.gradle` → **Java** — read from `java/`
   - `*.kt`, `*.kts`, `build.gradle.kts` → **Java** — Kotlin uses the Java SDK, read from `java/`
   - `*.scala`, `build.sbt` → **Java** — Scala uses the Java SDK, read from `java/`
   - `*.go`, `go.mod` → **Go** — read from `go/`
   - `*.rb`, `Gemfile` → **Ruby** — read from `ruby/`
   - `*.cs`, `*.csproj` → **C#** — read from `csharp/`
   - `*.php`, `composer.json` → **PHP** — read from `php/`

2. **如果检测到多种语言**（例如同时存在 Python 和 TypeScript 文件）：

2. **If multiple languages detected** (e.g., both Python and TypeScript files):

   - 检查用户当前文件或问题涉及哪种语言
   - 如果仍然不明确，询问："我检测到了 Python 和 TypeScript 文件。你使用哪种语言进行 Claude API 集成？"

   - Check which language the user's current file or question relates to
   - If still ambiguous, ask: "I detected both Python and TypeScript files. Which language are you using for the Claude API integration?"

3. **如果无法推断语言**（空项目、无源文件或不支持的语言）：

3. **If language can't be inferred** (empty project, no source files, or unsupported language):

   - 使用 AskUserQuestion 提供选项：Python、TypeScript、Java、Go、Ruby、cURL/原始 HTTP、C#、PHP
   - 如果 AskUserQuestion 不可用，默认展示 Python 示例并注明："展示 Python 示例。如需其他语言请告知。"

   - Use AskUserQuestion with options: Python, TypeScript, Java, Go, Ruby, cURL/raw HTTP, C#, PHP
   - If AskUserQuestion is unavailable, default to Python examples and note: "Showing Python examples. Let me know if you need a different language."

4. **如果检测到不支持的语言**（Rust、Swift、C++、Elixir 等）：

4. **If unsupported language detected** (Rust, Swift, C++, Elixir, etc.):

   - 建议使用 `curl/` 中的 cURL/原始 HTTP 示例，并说明可能存在社区 SDK
   - 提供 Python 或 TypeScript 示例作为参考实现

   - Suggest cURL/raw HTTP examples from `curl/` and note that community SDKs may exist
   - Offer to show Python or TypeScript examples as reference implementations

5. **如果用户需要 cURL/原始 HTTP 示例**，从 `curl/` 读取。

5. **If user needs cURL/raw HTTP examples**, read from `curl/`.

### 各语言功能支持

Language-Specific Feature Support

| 语言 | Tool Runner | Agent SDK | 备注 |
| ---------- | ----------- | --------- | ------------------------------------- |
| Python     | 是（beta） | 是 | 完整支持——`@beta_tool` 装饰器 |
| TypeScript | 是（beta） | 是 | 完整支持——`betaZodTool` + Zod |
| Java       | 是（beta） | 否 | 使用注解类的 Beta 工具调用 |
| Go         | 是（beta） | 否 | `toolrunner` 包中的 `BetaToolRunner` |
| Ruby       | 是（beta） | 否 | Beta 中的 `BaseTool` + `tool_runner` |
| cURL       | 不适用 | 不适用 | 原始 HTTP，无 SDK 功能 |
| C#         | 否 | 否 | 官方 SDK |
| PHP        | 否 | 否 | 官方 SDK |

| Language   | Tool Runner | Agent SDK | Notes                                 |
| ---------- | ----------- | --------- | ------------------------------------- |
| Python     | Yes (beta)  | Yes       | Full support — `@beta_tool` decorator |
| TypeScript | Yes (beta)  | Yes       | Full support — `betaZodTool` + Zod    |
| Java       | Yes (beta)  | No        | Beta tool use with annotated classes  |
| Go         | Yes (beta)  | No        | `BetaToolRunner` in `toolrunner` pkg  |
| Ruby       | Yes (beta)  | No        | `BaseTool` + `tool_runner` in beta    |
| cURL       | N/A         | N/A       | Raw HTTP, no SDK features             |
| C#         | No          | No        | Official SDK                          |
| PHP        | No          | No        | Official SDK                          |

---

## 应该使用哪个接入层？

Which Surface Should I Use?

> **从简单开始。** 默认使用满足需求的最简单层级。单次 API 调用和工作流可以处理大多数用例——只有当任务确实需要开放式的、模型驱动的探索时，才使用智能体。

> **Start simple.** Default to the simplest tier that meets your needs. Single API calls and workflows handle most use cases — only reach for agents when the task genuinely requires open-ended, model-driven exploration.

| 用例 | 层级 | 推荐接入层 | 原因 |
| ----------------------------------------------- | --------------- | ------------------------- | --------------------------------------- |
| 分类、摘要、提取、问答 | 单次 LLM 调用 | **Claude API** | 一次请求，一次响应 |
| 批量处理或嵌入 | 单次 LLM 调用 | **Claude API** | 专用端点 |
| 代码控制逻辑的多步骤流水线 | 工作流 | **Claude API + 工具调用** | 由你编排循环 |
| 使用自定义工具的智能体 | 智能体 | **Claude API + 工具调用** | 最大灵活性 |
| 具有文件/网络/终端访问的 AI 智能体 | 智能体 | **Agent SDK** | 内置工具、安全性和 MCP 支持 |
| 智能编码助手 | 智能体 | **Agent SDK** | 专为此用例设计 |
| 需要内置权限和护栏 | 智能体 | **Agent SDK** | 包含安全功能 |

| Use Case                                        | Tier            | Recommended Surface       | Why                                     |
| ----------------------------------------------- | --------------- | ------------------------- | --------------------------------------- |
| Classification, summarization, extraction, Q&A  | Single LLM call | **Claude API**            | One request, one response               |
| Batch processing or embeddings                  | Single LLM call | **Claude API**            | Specialized endpoints                   |
| Multi-step pipelines with code-controlled logic | Workflow        | **Claude API + tool use** | You orchestrate the loop                |
| Custom agent with your own tools                | Agent           | **Claude API + tool use** | Maximum flexibility                     |
| AI agent with file/web/terminal access          | Agent           | **Agent SDK**             | Built-in tools, safety, and MCP support |
| Agentic coding assistant                        | Agent           | **Agent SDK**             | Designed for this use case              |
| Want built-in permissions and guardrails        | Agent           | **Agent SDK**             | Safety features included                |

> **注意：** Agent SDK 适用于你需要开箱即用的内置文件/网络/终端工具、权限和 MCP 的场景。如果你想使用自己的工具构建智能体，Claude API 是正确的选择——使用 Tool Runner 自动处理循环，或使用手动循环进行细粒度控制（审批门控、自定义日志、条件执行）。

> **Note:** The Agent SDK is for when you want built-in file/web/terminal tools, permissions, and MCP out of the box. If you want to build an agent with your own tools, Claude API is the right choice — use the tool runner for automatic loop handling, or the manual loop for fine-grained control (approval gates, custom logging, conditional execution).

### 决策树

Decision Tree

```
你的应用需要什么？

1. 单次 LLM 调用（分类、摘要、提取、问答）
   └── Claude API——一次请求，一次响应

2. Claude 是否需要读写文件、浏览网页或运行 shell 命令
   来完成工作？（不是指：你的应用读取文件并传给 Claude——
   而是 Claude 本身是否需要发现和访问文件/网络/shell？）
   └── 是 → Agent SDK——内置工具，不需要重新实现
       示例："扫描代码库查找 bug"、"总结目录中的每个文件"、
             "使用子智能体查找 bug"、"通过网络搜索研究某个主题"

3. 工作流（多步骤，代码编排，使用你自己的工具）
   └── Claude API + 工具调用——你控制循环

4. 开放式智能体（模型自主决定轨迹，你自己的工具）
   └── Claude API 智能体循环（最大灵活性）
```

```
What does your application need?

1. Single LLM call (classification, summarization, extraction, Q&A)
   └── Claude API — one request, one response

2. Does Claude need to read/write files, browse the web, or run shell commands
   as part of its work? (Not: does your app read a file and hand it to Claude —
   does Claude itself need to discover and access files/web/shell?)
   └── Yes → Agent SDK — built-in tools, don't reimplement them
       Examples: "scan a codebase for bugs", "summarize every file in a directory",
                 "find bugs using subagents", "research a topic via web search"

3. Workflow (multi-step, code-orchestrated, with your own tools)
   └── Claude API with tool use — you control the loop

4. Open-ended agent (model decides its own trajectory, your own tools)
   └── Claude API agentic loop (maximum flexibility)
```

### 我应该构建智能体吗？

Should I Build an Agent?

在选择智能体层级之前，检查以下四个标准：

Before choosing the agent tier, check all four criteria:

- **复杂性** ——任务是否是多步骤的，且难以提前完全规定？（例如，"将这份设计文档转化为 PR" vs. "从这个 PDF 中提取标题"）
- **价值** ——产出是否值得更高的成本和延迟？
- **可行性** ——Claude 是否擅长这类任务？
- **错误成本** ——错误是否可以被捕获和恢复？（测试、审查、回滚）

- **Complexity** — Is the task multi-step and hard to fully specify in advance? (e.g., "turn this design doc into a PR" vs. "extract the title from this PDF")
- **Value** — Does the outcome justify higher cost and latency?
- **Viability** — Is Claude capable at this task type?
- **Cost of error** — Can errors be caught and recovered from? (tests, review, rollback)

如果对其中任何一项的回答是"否"，请停留在更简单的层级（单次调用或工作流）。

If the answer is "no" to any of these, stay at a simpler tier (single call or workflow).

---

## 架构

Architecture

一切都通过 `POST /v1/messages` 进行。工具和输出约束是这个单一端点的功能——而不是独立的 API。

Everything goes through `POST /v1/messages`. Tools and output constraints are features of this single endpoint — not separate APIs.

**用户定义的工具** ——你定义工具（通过装饰器、Zod schema 或原始 JSON），SDK 的 Tool Runner 负责调用 API、执行你的函数并循环直到 Claude 完成。如需完全控制，你可以手动编写循环。

**User-defined tools** — You define tools (via decorators, Zod schemas, or raw JSON), and the SDK's tool runner handles calling the API, executing your functions, and looping until Claude is done. For full control, you can write the loop manually.

**服务端工具** —— 在 Anthropic 基础设施上运行的 Anthropic 托管工具。代码执行完全在服务端进行（在 `tools` 中声明，Claude 自动运行代码）。Computer Use 可以是服务端托管或自托管。

**Server-side tools** — Anthropic-hosted tools that run on Anthropic's infrastructure. Code execution is fully server-side (declare it in `tools`, Claude runs code automatically). Computer use can be server-hosted or self-hosted.

**结构化输出** ——约束 Messages API 响应格式（`output_config.format`）和/或工具参数验证（`strict: true`）。推荐的方法是使用 `client.messages.parse()`，它会自动根据你的 schema 验证响应。注意：旧的 `output_format` 参数已弃用；请在 `messages.create()` 上使用 `output_config: {format: {...}}`。

**Structured outputs** — Constrains the Messages API response format (`output_config.format`) and/or tool parameter validation (`strict: true`). The recommended approach is `client.messages.parse()` which validates responses against your schema automatically. Note: the old `output_format` parameter is deprecated; use `output_config: {format: {...}}` on `messages.create()`.

**辅助端点** ——批量处理（`POST /v1/messages/batches`）、文件（`POST /v1/files`）和 Token 计数为 Messages API 请求提供支持。

**Supporting endpoints** — Batches (`POST /v1/messages/batches`), Files (`POST /v1/files`), and Token Counting feed into or support Messages API requests.

---

## 当前模型（缓存时间：2026-02-17）

Current Models (cached: 2026-02-17)

| 模型 | 模型 ID | 上下文 | 输入 $/1M | 输出 $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Opus 4.6   | `claude-opus-4-6`   | 200K（1M beta）| $5.00      | $25.00      |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K（1M beta）| $3.00      | $15.00      |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | 200K           | $1.00      | $5.00       |

| Model             | Model ID            | Context        | Input $/1M | Output $/1M |
| ----------------- | ------------------- | -------------- | ---------- | ----------- |
| Claude Opus 4.6   | `claude-opus-4-6`   | 200K (1M beta) | $5.00      | $25.00      |
| Claude Sonnet 4.6 | `claude-sonnet-4-6` | 200K (1M beta) | $3.00      | $15.00      |
| Claude Haiku 4.5  | `claude-haiku-4-5`  | 200K           | $1.00      | $5.00       |

**始终使用 `claude-opus-4-6`，除非用户明确指定了不同的模型。** 这一点不可商量。除非用户明确说"使用 sonnet"或"使用 haiku"，否则不要使用 `claude-sonnet-4-6`、`claude-sonnet-4-5` 或任何其他模型。绝不要为了节省成本而降级——那是用户的决定，不是你的。

**ALWAYS use `claude-opus-4-6` unless the user explicitly names a different model.** This is non-negotiable. Do not use `claude-sonnet-4-6`, `claude-sonnet-4-5`, or any other model unless the user literally says "use sonnet" or "use haiku". Never downgrade for cost — that's the user's decision, not yours.

**关键：仅使用上表中的确切模型 ID 字符串——它们本身就是完整的。不要追加日期后缀。** 例如，使用 `claude-sonnet-4-5`，而不是 `claude-sonnet-4-5-20250514` 或你可能从训练数据中回忆起的任何其他带日期后缀的变体。如果用户请求表中未列出的旧模型（例如"opus 4.5"、"sonnet 3.7"），请阅读 `shared/models.md` 获取确切 ID——这只是因为它们在你的训练数据截止日期之后发布。请放心，它们是真实的模型。

**CRITICAL: Use only the exact model ID strings from the table above — they are complete as-is. Do not append date suffixes.** For example, use `claude-sonnet-4-5`, never `claude-sonnet-4-5-20250514` or any other date-suffixed variant you might recall from training data. If the user requests an older model not in the table (e.g., "opus 4.5", "sonnet 3.7"), read `shared/models.md` for the exact ID — that just means they were released after your training data cutoff. Rest assured they are real models; we wouldn't mess with you like that.

---

## 思考与推理深度（快速参考）

Thinking & Effort (Quick Reference)

**Opus 4.6 —— 自适应思考（推荐）：** 使用 `thinking: {type: "adaptive"}`。Claude 会动态决定何时思考以及思考多深。不需要 `budget_tokens`——`budget_tokens` 在 Opus 4.6 和 Sonnet 4.6 上已弃用且不得使用。自适应思考还会自动启用交错思考（无需 beta 头部）。**当用户要求"扩展思考"、"思考预算"或 `budget_tokens` 时：始终使用 Opus 4.6 配合 `thinking: {type: "adaptive"}`。固定 token 思考预算的概念已被弃用——自适应思考取代了它。不要使用 `budget_tokens`，也不要切换到旧模型。**

**Opus 4.6 — Adaptive thinking (recommended):** Use `thinking: {type: "adaptive"}`. Claude dynamically decides when and how much to think. No `budget_tokens` needed — `budget_tokens` is deprecated on Opus 4.6 and Sonnet 4.6 and must not be used. Adaptive thinking also automatically enables interleaved thinking (no beta header needed). **When the user asks for "extended thinking", a "thinking budget", or `budget_tokens`: always use Opus 4.6 with `thinking: {type: "adaptive"}`. The concept of a fixed token budget for thinking is deprecated — adaptive thinking replaces it. Do NOT use `budget_tokens` and do NOT switch to an older model.**

**推理深度参数（GA，无需 beta 头部）：** 通过 `output_config: {effort: "low"|"medium"|"high"|"max"}` 控制思考深度和总体 token 消耗（在 `output_config` 内部，不是顶级参数）。默认值为 `high`（等同于省略它）。`max` 仅限 Opus 4.6。适用于 Opus 4.5、Opus 4.6 和 Sonnet 4.6。在 Sonnet 4.5 / Haiku 4.5 上会报错。结合自适应思考使用可获得最佳成本-质量权衡。对子智能体或简单任务使用 `low`；对最深层推理使用 `max`。

**Effort parameter (GA, no beta header):** Controls thinking depth and overall token spend via `output_config: {effort: "low"|"medium"|"high"|"max"}` (inside `output_config`, not top-level). Default is `high` (equivalent to omitting it). `max` is Opus 4.6 only. Works on Opus 4.5, Opus 4.6, and Sonnet 4.6. Will error on Sonnet 4.5 / Haiku 4.5. Combine with adaptive thinking for the best cost-quality tradeoffs. Use `low` for subagents or simple tasks; `max` for the deepest reasoning.

**Sonnet 4.6：** 支持自适应思考（`thinking: {type: "adaptive"}`）。`budget_tokens` 在 Sonnet 4.6 上已弃用——请改用自适应思考。

**Sonnet 4.6:** Supports adaptive thinking (`thinking: {type: "adaptive"}`). `budget_tokens` is deprecated on Sonnet 4.6 — use adaptive thinking instead.

**旧模型（仅在明确要求时使用）：** 如果用户明确要求 Sonnet 4.5 或其他旧模型，使用 `thinking: {type: "enabled", budget_tokens: N}`。`budget_tokens` 必须小于 `max_tokens`（最小值 1024）。不要仅仅因为用户提到 `budget_tokens` 就选择旧模型——请改用 Opus 4.6 配合自适应思考。

**Older models (only if explicitly requested):** If the user specifically asks for Sonnet 4.5 or another older model, use `thinking: {type: "enabled", budget_tokens: N}`. `budget_tokens` must be less than `max_tokens` (minimum 1024). Never choose an older model just because the user mentions `budget_tokens` — use Opus 4.6 with adaptive thinking instead.

---

## 上下文压缩（快速参考）

Compaction (Quick Reference)

**Beta 版，仅限 Opus 4.6。** 对于可能超出 200K 上下文窗口的长时间运行对话，启用服务端上下文压缩。当接近触发阈值（默认：150K token）时，API 会自动总结早期上下文。需要 beta 头部 `compact-2026-01-12`。

**Beta, Opus 4.6 only.** For long-running conversations that may exceed the 200K context window, enable server-side compaction. The API automatically summarizes earlier context when it approaches the trigger threshold (default: 150K tokens). Requires beta header `compact-2026-01-12`.

**关键：** 在每一轮对话中将 `response.content`（而不仅仅是文本）追加回你的消息列表。响应中的压缩块必须被保留——API 在下一次请求中使用它们来替换已压缩的历史记录。仅提取文本字符串并追加会静默丢失压缩状态。

**Critical:** Append `response.content` (not just the text) back to your messages on every turn. Compaction blocks in the response must be preserved — the API uses them to replace the compacted history on the next request. Extracting only the text string and appending that will silently lose the compaction state.

参见 `{lang}/claude-api/README.md`（上下文压缩部分）获取代码示例。通过 WebFetch 在 `shared/live-sources.md` 中获取完整文档。

See `{lang}/claude-api/README.md` (Compaction section) for code examples. Full docs via WebFetch in `shared/live-sources.md`.

---

## 阅读指南

Reading Guide

检测到语言后，根据用户需要阅读相关文件：

After detecting the language, read the relevant files based on what the user needs:

### 快速任务参考

Quick Task Reference

**单次文本分类/摘要/提取/问答：**
→ 仅阅读 `{lang}/claude-api/README.md`

**Single text classification/summarization/extraction/Q&A:**
→ Read only `{lang}/claude-api/README.md`

**聊天 UI 或实时响应展示：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**Chat UI or real-time response display:**
→ Read `{lang}/claude-api/README.md` + `{lang}/claude-api/streaming.md`

**长时间运行的对话（可能超出上下文窗口）：**
→ 阅读 `{lang}/claude-api/README.md`——参见上下文压缩部分

**Long-running conversations (may exceed context window):**
→ Read `{lang}/claude-api/README.md` — see Compaction section

**函数调用 / 工具使用 / 智能体：**
→ 阅读 `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**Function calling / tool use / agents:**
→ Read `{lang}/claude-api/README.md` + `shared/tool-use-concepts.md` + `{lang}/claude-api/tool-use.md`

**批量处理（非延迟敏感）：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**Batch processing (non-latency-sensitive):**
→ Read `{lang}/claude-api/README.md` + `{lang}/claude-api/batches.md`

**跨多个请求上传文件：**
→ 阅读 `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**File uploads across multiple requests:**
→ Read `{lang}/claude-api/README.md` + `{lang}/claude-api/files-api.md`

**带内置工具的智能体（文件/网络/终端）：**
→ 阅读 `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

**Agent with built-in tools (file/web/terminal):**
→ Read `{lang}/agent-sdk/README.md` + `{lang}/agent-sdk/patterns.md`

### Claude API（完整文件参考）

Claude API (Full File Reference)

阅读**语言专属的 Claude API 文件夹**（`{language}/claude-api/`）：

Read the **language-specific Claude API folder** (`{language}/claude-api/`):

1. **`{language}/claude-api/README.md`** —— **首先阅读此文件。** 安装、快速开始、常见模式、错误处理。
2. **`shared/tool-use-concepts.md`** ——当用户需要函数调用、代码执行、记忆或结构化输出时阅读。涵盖概念基础。
3. **`{language}/claude-api/tool-use.md`** ——阅读语言专属的工具使用代码示例（Tool Runner、手动循环、代码执行、记忆、结构化输出）。
4. **`{language}/claude-api/streaming.md`** ——在构建聊天 UI 或增量显示响应的界面时阅读。
5. **`{language}/claude-api/batches.md`** ——在离线处理大量请求（非延迟敏感）时阅读。异步运行，成本降低 50%。
6. **`{language}/claude-api/files-api.md`** ——在跨多个请求发送相同文件而无需重新上传时阅读。
7. **`shared/error-codes.md`** ——在调试 HTTP 错误或实现错误处理时阅读。
8. **`shared/live-sources.md`** ——用于获取最新官方文档的 WebFetch URL。

1. **`{language}/claude-api/README.md`** — **Read this first.** Installation, quick start, common patterns, error handling.
2. **`shared/tool-use-concepts.md`** — Read when the user needs function calling, code execution, memory, or structured outputs. Covers conceptual foundations.
3. **`{language}/claude-api/tool-use.md`** — Read for language-specific tool use code examples (tool runner, manual loop, code execution, memory, structured outputs).
4. **`{language}/claude-api/streaming.md`** — Read when building chat UIs or interfaces that display responses incrementally.
5. **`{language}/claude-api/batches.md`** — Read when processing many requests offline (not latency-sensitive). Runs asynchronously at 50% cost.
6. **`{language}/claude-api/files-api.md`** — Read when sending the same file across multiple requests without re-uploading.
7. **`shared/error-codes.md`** — Read when debugging HTTP errors or implementing error handling.
8. **`shared/live-sources.md`** — WebFetch URLs for fetching the latest official documentation.

> **注意：** 对于 Java、Go、Ruby、C#、PHP 和 cURL——它们各有一个涵盖所有基础内容的单一文件。根据需要阅读该文件加上 `shared/tool-use-concepts.md` 和 `shared/error-codes.md`。

> **Note:** For Java, Go, Ruby, C#, PHP, and cURL — these have a single file each covering all basics. Read that file plus `shared/tool-use-concepts.md` and `shared/error-codes.md` as needed.

### Agent SDK

Agent SDK

阅读**语言专属的 Agent SDK 文件夹**（`{language}/agent-sdk/`）。Agent SDK 仅适用于 **Python 和 TypeScript**。

Read the **language-specific Agent SDK folder** (`{language}/agent-sdk/`). Agent SDK is available for **Python and TypeScript only**.

1. **`{language}/agent-sdk/README.md`** ——安装、快速开始、内置工具、权限、MCP、钩子。
2. **`{language}/agent-sdk/patterns.md`** ——自定义工具、钩子、子智能体、MCP 集成、会话恢复。
3. **`shared/live-sources.md`** ——当前 Agent SDK 文档的 WebFetch URL。

1. **`{language}/agent-sdk/README.md`** — Installation, quick start, built-in tools, permissions, MCP, hooks.
2. **`{language}/agent-sdk/patterns.md`** — Custom tools, hooks, subagents, MCP integration, session resumption.
3. **`shared/live-sources.md`** — WebFetch URLs for current Agent SDK docs.

---

## 何时使用 WebFetch

When to Use WebFetch

在以下情况下使用 WebFetch 获取最新文档：

Use WebFetch to get the latest documentation when:

- 用户要求"最新"或"当前"信息
- 缓存数据似乎不正确
- 用户询问此处未涵盖的功能

- User asks for "latest" or "current" information
- Cached data seems incorrect
- User asks about features not covered here

实时文档 URL 在 `shared/live-sources.md` 中。

Live documentation URLs are in `shared/live-sources.md`.

## 常见陷阱

Common Pitfalls

- 在将文件或内容传递给 API 时，不要截断输入。如果内容太长无法放入上下文窗口，请通知用户并讨论选项（分块、摘要等），而不是静默截断。

- Don't truncate inputs when passing files or content to the API. If the content is too long to fit in the context window, notify the user and discuss options (chunking, summarization, etc.) rather than silently truncating.

- **Opus 4.6 / Sonnet 4.6 思考：** 使用 `thinking: {type: "adaptive"}`——不要使用 `budget_tokens`（在 Opus 4.6 和 Sonnet 4.6 上已弃用）。对于旧模型，`budget_tokens` 必须小于 `max_tokens`（最小值 1024）。如果设置错误将会抛出错误。

- **Opus 4.6 / Sonnet 4.6 thinking:** Use `thinking: {type: "adaptive"}` — do NOT use `budget_tokens` (deprecated on both Opus 4.6 and Sonnet 4.6). For older models, `budget_tokens` must be less than `max_tokens` (minimum 1024). This will throw an error if you get it wrong.

- **Opus 4.6 预填充已移除：** 助手消息预填充（最后一轮助手消息预填充）在 Opus 4.6 上会返回 400 错误。请改用结构化输出（`output_config.format`）或系统提示指令来控制响应格式。

- **Opus 4.6 prefill removed:** Assistant message prefills (last-assistant-turn prefills) return a 400 error on Opus 4.6. Use structured outputs (`output_config.format`) or system prompt instructions to control response format instead.

- **128K 输出 token：** Opus 4.6 支持最高 128K 的 `max_tokens`，但 SDK 要求对大 `max_tokens` 使用流式传输以避免 HTTP 超时。使用 `.stream()` 配合 `.get_final_message()` / `.finalMessage()`。

- **128K output tokens:** Opus 4.6 supports up to 128K `max_tokens`, but the SDKs require streaming for large `max_tokens` to avoid HTTP timeouts. Use `.stream()` with `.get_final_message()` / `.finalMessage()`.

- **工具调用 JSON 解析（Opus 4.6）：** Opus 4.6 可能在工具调用的 `input` 字段中产生不同的 JSON 字符串转义（例如 Unicode 或正斜杠转义）。始终使用 `json.loads()` / `JSON.parse()` 解析工具输入——不要对序列化的输入进行原始字符串匹配。

- **Tool call JSON parsing (Opus 4.6):** Opus 4.6 may produce different JSON string escaping in tool call `input` fields (e.g., Unicode or forward-slash escaping). Always parse tool inputs with `json.loads()` / `JSON.parse()` — never do raw string matching on the serialized input.

- **结构化输出（所有模型）：** 使用 `output_config: {format: {...}}` 代替 `messages.create()` 上已弃用的 `output_format` 参数。这是一个通用的 API 变更，而非 4.6 特有的。

- **Structured outputs (all models):** Use `output_config: {format: {...}}` instead of the deprecated `output_format` parameter on `messages.create()`. This is a general API change, not 4.6-specific.

- **不要重新实现 SDK 功能：** SDK 提供了高级辅助方法——使用它们而不是从头构建。具体来说：使用 `stream.finalMessage()` 而不是用 `new Promise()` 包装 `.on()` 事件；使用类型化的异常类（`Anthropic.RateLimitError` 等）而不是字符串匹配错误消息；使用 SDK 类型（`Anthropic.MessageParam`、`Anthropic.Tool`、`Anthropic.Message` 等）而不是重新定义等效接口。

- **Don't reimplement SDK functionality:** The SDK provides high-level helpers — use them instead of building from scratch. Specifically: use `stream.finalMessage()` instead of wrapping `.on()` events in `new Promise()`; use typed exception classes (`Anthropic.RateLimitError`, etc.) instead of string-matching error messages; use SDK types (`Anthropic.MessageParam`, `Anthropic.Tool`, `Anthropic.Message`, etc.) instead of redefining equivalent interfaces.

- **不要为 SDK 数据结构定义自定义类型：** SDK 导出了所有 API 对象的类型。使用 `Anthropic.MessageParam` 表示消息，`Anthropic.Tool` 表示工具定义，`Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam` 表示工具结果，`Anthropic.Message` 表示响应。定义自己的 `interface ChatMessage { role: string; content: unknown }` 会重复 SDK 已提供的内容并丧失类型安全。

- **Don't define custom types for SDK data structures:** The SDK exports types for all API objects. Use `Anthropic.MessageParam` for messages, `Anthropic.Tool` for tool definitions, `Anthropic.ToolUseBlock` / `Anthropic.ToolResultBlockParam` for tool results, `Anthropic.Message` for responses. Defining your own `interface ChatMessage { role: string; content: unknown }` duplicates what the SDK already provides and loses type safety.

- **报告和文档输出：** 对于生成报告、文档或可视化的任务，代码执行沙箱预装了 `python-docx`、`python-pptx`、`matplotlib`、`pillow` 和 `pypdf`。Claude 可以生成格式化文件（DOCX、PDF、图表）并通过 Files API 返回——对于"报告"或"文档"类型的请求，考虑使用这种方式而不是纯文本 stdout 输出。

- **Report and document output:** For tasks that produce reports, documents, or visualizations, the code execution sandbox has `python-docx`, `python-pptx`, `matplotlib`, `pillow`, and `pypdf` pre-installed. Claude can generate formatted files (DOCX, PDF, charts) and return them via the Files API — consider this for "report" or "document" type requests instead of plain stdout text.
