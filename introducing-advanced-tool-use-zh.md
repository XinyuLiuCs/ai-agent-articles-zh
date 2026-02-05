# Claude 开发者平台的高级工具使用

Introducing advanced tool use on the Claude Developer Platform

发布日期：2025年11月24日

Published Nov 24, 2025

我们新增了三个测试版功能，让 Claude 能够动态发现、学习和执行工具。以下是它们的工作原理。

We've added three new beta features that let Claude discover, learn, and execute tools dynamically. Here's how they work.

AI 智能体的未来是模型能够无缝协作使用成百上千个工具。一个集成了 git 操作、文件操作、包管理器、测试框架和部署流水线的 IDE 助手。一个同时连接 Slack、GitHub、Google Drive、Jira、公司数据库和数十个 MCP 服务器的运维协调器。

The future of AI agents is one where models work seamlessly across hundreds or thousands of tools. An IDE assistant that integrates git operations, file manipulation, package managers, testing frameworks, and deployment pipelines. An operations coordinator that connects Slack, GitHub, Google Drive, Jira, company databases, and dozens of MCP servers simultaneously.

要构建有效的智能体，它们需要能够处理无限的工具库，而无需预先将每个定义都塞进上下文。我们关于将代码执行与 MCP 结合使用的博客文章讨论了工具结果和定义有时会在智能体读取请求之前消耗 50,000+ 个标记。智能体应该按需发现和加载工具，仅保留与当前任务相关的内容。

To build effective agents, they need to work with unlimited tool libraries without stuffing every definition into context upfront. Our blog article on using code execution with MCP discussed how tool results and definitions can sometimes consume 50,000+ tokens before an agent reads a request. Agents should discover and load tools on-demand, keeping only what's relevant for the current task.

智能体还需要能够从代码中调用工具。当使用自然语言工具调用时，每次调用都需要完整的推理过程，中间结果会堆积在上下文中，无论它们是否有用。代码天然适合编排逻辑，例如循环、条件判断和数据转换。智能体需要根据手头任务在代码执行和推理之间灵活选择。

Agents also need the ability to call tools from code. When using natural language tool calling, each invocation requires a full inference pass, and intermediate results pile up in context whether they're useful or not. Code is a natural fit for orchestration logic, such as loops, conditionals, and data transformations. Agents need the flexibility to choose between code execution and inference based on the task at hand.

智能体还需要从示例中学习正确的工具使用方法，而不仅仅是从模式定义中学习。JSON 模式定义了结构上的有效性，但无法表达使用模式：何时包含可选参数、哪些组合有意义，或你的 API 期望什么约定。

Agents also need to learn correct tool usage from examples, not just schema definitions. JSON schemas define what's structurally valid, but can't express usage patterns: when to include optional parameters, which combinations make sense, or what conventions your API expects.

今天，我们发布了三个功能来实现这一目标：

Today, we're releasing three features that make this possible:

**工具搜索工具**，允许 Claude 使用搜索工具访问数千个工具而不消耗其上下文窗口

Tool Search Tool, which allows Claude to use search tools to access thousands of tools without consuming its context window

**编程式工具调用**，允许 Claude 在代码执行环境中调用工具，减少对模型上下文窗口的影响

Programmatic Tool Calling, which allows Claude to invoke tools in a code execution environment reducing the impact on the model's context window

**工具使用示例**，为演示如何有效使用给定工具提供通用标准

Tool Use Examples, which provides a universal standard for demonstrating how to effectively use a given tool

在内部测试中，我们发现这些功能帮助我们构建了传统工具使用模式无法实现的东西。例如，Claude for Excel 使用编程式工具调用来读取和修改包含数千行的电子表格，而不会使模型的上下文窗口过载。

In internal testing, we've found these features have helped us build things that wouldn't have been possible with conventional tool use patterns. For example, Claude for Excel uses Programmatic Tool Calling to read and modify spreadsheets with thousands of rows without overloading the model's context window.

根据我们的经验，我们相信这些功能为你使用 Claude 构建的内容开辟了新的可能性。

Based on our experience, we believe these features open up new possibilities for what you can build with Claude.

## 工具搜索工具

Tool Search Tool

### 挑战

The challenge

MCP 工具定义提供了重要的上下文，但随着连接的服务器增多，这些标记会累积起来。考虑一个五服务器设置：

MCP tool definitions provide important context, but as more servers connect, those tokens can add up. Consider a five-server setup:

GitHub：35 个工具（约 26K 标记）
Slack：11 个工具（约 21K 标记）
Sentry：5 个工具（约 3K 标记）
Grafana：5 个工具（约 3K 标记）
Splunk：2 个工具（约 2K 标记）

GitHub: 35 tools (~26K tokens)
Slack: 11 tools (~21K tokens)
Sentry: 5 tools (~3K tokens)
Grafana: 5 tools (~3K tokens)
Splunk: 2 tools (~2K tokens)

这是 58 个工具，在对话开始之前就消耗了约 55K 个标记。再添加更多服务器，如 Jira（仅此一项就使用约 17K 个标记），你很快就会接近 100K+ 的标记开销。在 Anthropic，我们看到工具定义在优化之前消耗了 134K 个标记。

That's 58 tools consuming approximately 55K tokens before the conversation even starts. Add more servers like Jira (which alone uses ~17K tokens) and you're quickly approaching 100K+ token overhead. At Anthropic, we've seen tool definitions consume 134K tokens before optimization.

但标记成本不是唯一的问题。**最常见的失败是错误的工具选择和不正确的参数，特别是当工具具有类似名称时**，如 notification-send-user 与 notification-send-channel。

But token cost isn't the only issue. The most common failures are wrong tool selection and incorrect parameters, especially when tools have similar names like notification-send-user vs. notification-send-channel.

### 我们的解决方案

Our solution

工具搜索工具不是预先加载所有工具定义，而是按需发现工具。Claude 只看到它当前任务实际需要的工具。

Instead of loading all tool definitions upfront, the Tool Search Tool discovers tools on-demand. Claude only sees the tools it actually needs for the current task.

**工具搜索工具示意图**

Tool Search Tool diagram

与 Claude 的传统方法（122,800 个标记）相比，工具搜索工具保留了 191,300 个上下文标记。

Tool Search Tool preserves 191,300 tokens of context compared to 122,800 with Claude's traditional approach.

传统方法：

Traditional approach:

预先加载所有工具定义（50+ 个 MCP 工具约 72K 标记）
对话历史和系统提示争夺剩余空间
在任何工作开始之前，总上下文消耗：约 77K 标记

All tool definitions loaded upfront (~72K tokens for 50+ MCP tools)
Conversation history and system prompt compete for remaining space
Total context consumption: ~77K tokens before any work begins

使用工具搜索工具：

With the Tool Search Tool:

只有工具搜索工具预先加载（约 500 个标记）
根据需要按需发现工具（3-5 个相关工具，约 3K 标记）
总上下文消耗：约 8.7K 标记，保留了 95% 的上下文窗口

Only the Tool Search Tool loaded upfront (~500 tokens)
Tools discovered on-demand as needed (3-5 relevant tools, ~3K tokens)
Total context consumption: ~8.7K tokens, preserving 95% of context window

这代表标记使用量减少了 85%，同时保持对完整工具库的访问。内部测试显示，在使用大型工具库时，MCP 评估的准确性有了显著提高。启用工具搜索工具后，Opus 4 从 49% 提高到 74%，Opus 4.5 从 79.5% 提高到 88.1%。

This represents an 85% reduction in token usage while maintaining access to your full tool library. Internal testing showed significant accuracy improvements on MCP evaluations when working with large tool libraries. Opus 4 improved from 49% to 74%, and Opus 4.5 improved from 79.5% to 88.1% with Tool Search Tool enabled.

### 工具搜索工具的工作原理

How the Tool Search Tool works

工具搜索工具让 Claude 动态发现工具，而不是预先加载所有定义。你将所有工具定义提供给 API，但使用 `defer_loading: true` 标记工具，使其可按需发现。延迟加载的工具最初不会加载到 Claude 的上下文中。Claude 只看到工具搜索工具本身以及任何设置了 `defer_loading: false` 的工具（你最关键、最常用的工具）。

The Tool Search Tool lets Claude dynamically discover tools instead of loading all definitions upfront. You provide all your tool definitions to the API, but mark tools with defer_loading: true to make them discoverable on-demand. Deferred tools aren't loaded into Claude's context initially. Claude only sees the Tool Search Tool itself plus any tools with defer_loading: false (your most critical, frequently-used tools).

当 Claude 需要特定功能时，它会搜索相关工具。工具搜索工具返回对匹配工具的引用，这些引用会在 Claude 的上下文中扩展为完整定义。

When Claude needs specific capabilities, it searches for relevant tools. The Tool Search Tool returns references to matching tools, which get expanded into full definitions in Claude's context.

例如，如果 Claude 需要与 GitHub 交互，它会搜索"github"，只有 `github.createPullRequest` 和 `github.listIssues` 会被加载——而不是来自 Slack、Jira 和 Google Drive 的其他 50+ 个工具。

For example, if Claude needs to interact with GitHub, it searches for "github," and only github.createPullRequest and github.listIssues get loaded—not your other 50+ tools from Slack, Jira, and Google Drive.

这样，Claude 可以访问你的完整工具库，同时只为它实际需要的工具支付标记成本。

This way, Claude has access to your full tool library while only paying the token cost for tools it actually needs.

**提示缓存注意事项：工具搜索工具不会破坏提示缓存，因为延迟加载的工具完全从初始提示中排除。它们只在 Claude 搜索后才添加到上下文中，因此你的系统提示和核心工具定义仍然可以缓存。**

Prompt caching note: Tool Search Tool doesn't break prompt caching because deferred tools are excluded from the initial prompt entirely. They're only added to context after Claude searches for them, so your system prompt and core tool definitions remain cacheable.

实现：

Implementation:

```json
{
  "tools": [
    // 包含工具搜索工具（正则表达式、BM25 或自定义）
    {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},

    // 标记工具以进行按需发现
    {
      "name": "github.createPullRequest",
      "description": "创建拉取请求",
      "input_schema": {...},
      "defer_loading": true
    }
    // ... 数百个带有 defer_loading: true 的延迟加载工具
  ]
}
```

{
  "tools": [
    // Include a tool search tool (regex, BM25, or custom)
    {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},

    // Mark tools for on-demand discovery
    {
      "name": "github.createPullRequest",
      "description": "Create a pull request",
      "input_schema": {...},
      "defer_loading": true
    }
    // ... hundreds more deferred tools with defer_loading: true
  ]
}

对于 MCP 服务器，你可以延迟加载整个服务器，同时保持特定高使用率工具的加载：

For MCP servers, you can defer loading entire servers while keeping specific high-use tools loaded:

```json
{
  "type": "mcp_toolset",
  "mcp_server_name": "google-drive",
  "default_config": {"defer_loading": true}, # 延迟加载整个服务器
  "configs": {
    "search_files": {
      "defer_loading": false
    }  // 保持最常用的工具加载
  }
}
```

{
  "type": "mcp_toolset",
  "mcp_server_name": "google-drive",
  "default_config": {"defer_loading": true}, # defer loading the entire server
  "configs": {
    "search_files": {
"defer_loading": false
    }  // Keep most used tool loaded
  }
}

Claude 开发者平台提供了开箱即用的基于正则表达式和 BM25 的搜索工具，但你也可以使用嵌入或其他策略实现自定义搜索工具。

The Claude Developer Platform provides regex-based and BM25-based search tools out of the box, but you can also implement custom search tools using embeddings or other strategies.

### 何时使用工具搜索工具

When to use the Tool Search Tool

像任何架构决策一样，启用工具搜索工具涉及权衡。该功能在工具调用之前添加了搜索步骤，因此当上下文节省和准确性改进超过额外延迟时，它能提供最佳投资回报率。

Like any architectural decision, enabling the Tool Search Tool involves trade-offs. The feature adds a search step before tool invocation, so it delivers the best ROI when the context savings and accuracy improvements outweigh additional latency.

适用场景：

Use it when:

工具定义消耗 >10K 标记
遇到工具选择准确性问题
构建具有多个服务器的 MCP 驱动系统
10+ 个可用工具

Tool definitions consuming >10K tokens
Experiencing tool selection accuracy issues
Building MCP-powered systems with multiple servers
10+ tools available

不太有益的场景：

Less beneficial when:

小型工具库（<10 个工具）
所有工具在每个会话中都频繁使用
工具定义紧凑

Small tool library (<10 tools)
All tools used frequently in every session
Tool definitions are compact

## 编程式工具调用

Programmatic Tool Calling

### 挑战

The challenge

随着工作流变得更加复杂，传统工具调用会产生两个根本问题：

Traditional tool calling creates two fundamental problems as workflows become more complex:

**中间结果导致的上下文污染**：当 Claude 分析 10MB 日志文件以查找错误模式时，整个文件会进入其上下文窗口，即使 Claude 只需要错误频率的摘要。当跨多个表获取客户数据时，每条记录都会在上下文中累积，无论是否相关。这些中间结果消耗了大量标记预算，并且可能将重要信息完全推出上下文窗口。

Context pollution from intermediate results: When Claude analyzes a 10MB log file for error patterns, the entire file enters its context window, even though Claude only needs a summary of error frequencies. When fetching customer data across multiple tables, every record accumulates in context regardless of relevance. These intermediate results consume massive token budgets and can push important information out of the context window entirely.

**推理开销和手动综合**：每次工具调用都需要完整的模型推理过程。在接收结果后，Claude 必须"目测"数据以提取相关信息，推理各部分如何拼合，并决定下一步做什么——所有这些都通过自然语言处理。一个五工具工作流意味着五次推理过程加上 Claude 解析每个结果、比较值和综合结论。这既缓慢又容易出错。

Inference overhead and manual synthesis: Each tool call requires a full model inference pass. After receiving results, Claude must "eyeball" the data to extract relevant information, reason about how pieces fit together, and decide what to do next—all through natural language processing. A five tool workflow means five inference passes plus Claude parsing each result, comparing values, and synthesizing conclusions. This is both slow and error-prone.

### 我们的解决方案

Our solution

编程式工具调用使 Claude 能够通过代码而不是通过单独的 API 往返来编排工具。Claude 不是一次请求一个工具并将每个结果返回到其上下文，而是**编写调用多个工具、处理其输出并控制哪些信息实际进入其上下文窗口的代码**。

Programmatic Tool Calling enables Claude to orchestrate tools through code rather than through individual API round-trips. Instead of Claude requesting tools one at a time with each result being returned to its context, Claude writes code that calls multiple tools, processes their outputs, and controls what information actually enters its context window.

Claude 擅长编写代码，通过让它用 Python 而不是通过自然语言工具调用来表达编排逻辑，你可以获得更可靠、更精确的控制流。循环、条件判断、数据转换和错误处理在代码中都是显式的，而不是隐含在 Claude 的推理中。

Claude excels at writing code and by letting it express orchestration logic in Python rather than through natural language tool invocations, you get more reliable, precise control flow. Loops, conditionals, data transformations, and error handling are all explicit in code rather than implicit in Claude's reasoning.

### 示例：预算合规检查

Example: Budget compliance check

考虑一个常见的业务任务："哪些团队成员超出了他们的第三季度差旅预算？"

Consider a common business task: "Which team members exceeded their Q3 travel budget?"

你有三个可用工具：

You have three tools available:

`get_team_members(department)` - 返回带有 ID 和级别的团队成员列表
`get_expenses(user_id, quarter)` - 返回用户的费用明细项
`get_budget_by_level(level)` - 返回员工级别的预算限额

get_team_members(department) - Returns team member list with IDs and levels
get_expenses(user_id, quarter) - Returns expense line items for a user
get_budget_by_level(level) - Returns budget limits for an employee level

传统方法：

Traditional approach:

1. 获取团队成员 → 20 人
2. 对于每个人，获取他们的第三季度费用 → 20 次工具调用，每次返回 50-100 个明细项（航班、酒店、餐饮、收据）
3. 按员工级别获取预算限额
4. 所有这些都进入 Claude 的上下文：2,000+ 个费用明细项（50 KB+）
5. Claude 手动汇总每个人的费用，查找他们的预算，将费用与预算限额进行比较
6. 更多到模型的往返，显著的上下文消耗

Fetch team members → 20 people
For each person, fetch their Q3 expenses → 20 tool calls, each returning 50-100 line items (flights, hotels, meals, receipts)
Fetch budget limits by employee level
All of this enters Claude's context: 2,000+ expense line items (50 KB+)
Claude manually sums each person's expenses, looks up their budget, compares expenses against budget limits
More round-trips to the model, significant context consumption

使用编程式工具调用：

With Programmatic Tool Calling:

每个工具结果不是返回给 Claude，而是 Claude 编写一个编排整个工作流的 Python 脚本。脚本在代码执行工具（沙盒环境）中运行，当需要你的工具结果时会暂停。当你通过 API 返回工具结果时，它们由脚本处理，而不是由模型消耗。脚本继续执行，Claude 只看到最终输出。

Instead of each tool result returning to Claude, Claude writes a Python script that orchestrates the entire workflow. The script runs in the Code Execution tool (a sandboxed environment), pausing when it needs results from your tools. When you return tool results via the API, they're processed by the script rather than consumed by the model. The script continues executing, and Claude only sees the final output.

**编程式工具调用流程**

Programmatic tool calling flow

编程式工具调用使 Claude 能够通过代码而不是通过单独的 API 往返来编排工具，允许并行执行工具。

Programmatic Tool Calling enables Claude to orchestrate tools through code rather than through individual API round-trips, allowing for parallel tool execution.

这是 Claude 为预算合规任务编写的编排代码：

Here's what Claude's orchestration code looks like for the budget compliance task:

```python
team = await get_team_members("engineering")

# 为每个唯一级别获取预算
levels = list(set(m["level"] for m in team))
budget_results = await asyncio.gather(*[
    get_budget_by_level(level) for level in levels
])

# 创建查找字典：{"junior": budget1, "senior": budget2, ...}
budgets = {level: budget for level, budget in zip(levels, budget_results)}

# 并行获取所有费用
expenses = await asyncio.gather(*[
    get_expenses(m["id"], "Q3") for m in team
])

# 查找超出差旅预算的员工
exceeded = []
for member, exp in zip(team, expenses):
    budget = budgets[member["level"]]
    total = sum(e["amount"] for e in exp)
    if total > budget["travel_limit"]:
        exceeded.append({
            "name": member["name"],
            "spent": total,
            "limit": budget["travel_limit"]
        })

print(json.dumps(exceeded))
```

team = await get_team_members("engineering")

# Fetch budgets for each unique level
levels = list(set(m["level"] for m in team))
budget_results = await asyncio.gather(*[
    get_budget_by_level(level) for level in levels
])

# Create a lookup dictionary: {"junior": budget1, "senior": budget2, ...}
budgets = {level: budget for level, budget in zip(levels, budget_results)}

# Fetch all expenses in parallel
expenses = await asyncio.gather(*[
    get_expenses(m["id"], "Q3") for m in team
])

# Find employees who exceeded their travel budget
exceeded = []
for member, exp in zip(team, expenses):
    budget = budgets[member["level"]]
    total = sum(e["amount"] for e in exp)
    if total > budget["travel_limit"]:
        exceeded.append({
            "name": member["name"],
            "spent": total,
            "limit": budget["travel_limit"]
        })

print(json.dumps(exceeded))

Claude 的上下文只接收最终结果：超出预算的两三个人。2,000+ 个明细项、中间汇总和预算查找不会影响 Claude 的上下文，将消耗从 200KB 的原始费用数据减少到仅 1KB 的结果。

Claude's context receives only the final result: the two to three people who exceeded their budget. The 2,000+ line items, the intermediate sums, and the budget lookups do not affect Claude's context, reducing consumption from 200KB of raw expense data to just 1KB of results.

效率提升是实质性的：

The efficiency gains are substantial:

**标记节省**：通过将中间结果保留在 Claude 的上下文之外，PTC 显著降低了标记消耗。在复杂研究任务上，平均使用量从 43,588 个标记降至 27,297 个标记，减少了 37%。

Token savings: By keeping intermediate results out of Claude's context, PTC dramatically reduces token consumption. Average usage dropped from 43,588 to 27,297 tokens, a 37% reduction on complex research tasks.

**减少延迟**：每次 API 往返都需要模型推理（数百毫秒到数秒）。当 Claude 在单个代码块中编排 20+ 次工具调用时，你消除了 19+ 次推理过程。API 处理工具执行而无需每次都返回到模型。

Reduced latency: Each API round-trip requires model inference (hundreds of milliseconds to seconds). When Claude orchestrates 20+ tool calls in a single code block, you eliminate 19+ inference passes. The API handles tool execution without returning to the model each time.

**提高准确性**：通过编写显式编排逻辑，Claude 在自然语言中处理多个工具结果时犯的错误更少。内部知识检索从 25.6% 提高到 28.5%；GIA 基准从 46.5% 提高到 51.2%。

Improved accuracy: By writing explicit orchestration logic, Claude makes fewer errors than when juggling multiple tool results in natural language. Internal knowledge retrieval improved from 25.6% to 28.5%; GIA benchmarks from 46.5% to 51.2%.

生产工作流涉及混乱的数据、条件逻辑和需要扩展的操作。编程式工具调用让 Claude 以编程方式处理这种复杂性，同时将其注意力集中在可操作的结果上，而不是原始数据处理上。

Production workflows involve messy data, conditional logic, and operations that need to scale. Programmatic Tool Calling lets Claude handle that complexity programmatically while keeping its focus on actionable results rather than raw data processing.

### 编程式工具调用的工作原理

How Programmatic Tool Calling works

**1. 将工具标记为可从代码调用**

1. Mark tools as callable from code

将 `code_execution` 添加到工具中，并将 `allowed_callers` 设置为选择性启用编程执行的工具：

Add code_execution to tools, and set allowed_callers to opt-in tools for programmatic execution:

```json
{
  "tools": [
    {
      "type": "code_execution_20250825",
      "name": "code_execution"
    },
    {
      "name": "get_team_members",
      "description": "获取部门的所有成员...",
      "input_schema": {...},
      "allowed_callers": ["code_execution_20250825"] # 选择性启用编程式工具调用
    },
    {
      "name": "get_expenses",
     ...
    },
    {
      "name": "get_budget_by_level",
    ...
    }
  ]
}
```

{
  "tools": [
    {
      "type": "code_execution_20250825",
      "name": "code_execution"
    },
    {
      "name": "get_team_members",
      "description": "Get all members of a department...",
      "input_schema": {...},
      "allowed_callers": ["code_execution_20250825"] # opt-in to programmatic tool calling
    },
    {
      "name": "get_expenses",
     ...
    },
    {
      "name": "get_budget_by_level",
    ...
    }
  ]
}

API 将这些工具定义转换为 Claude 可以调用的 Python 函数。

The API converts these tool definitions into Python functions that Claude can call.

**2. Claude 编写编排代码**

2. Claude writes orchestration code

Claude 不是一次请求一个工具，而是生成 Python 代码：

Instead of requesting tools one at a time, Claude generates Python code:

```json
{
  "type": "server_tool_use",
  "id": "srvtoolu_abc",
  "name": "code_execution",
  "input": {
    "code": "team = get_team_members('engineering')\n..." # 上面的代码示例
  }
}
```

{
  "type": "server_tool_use",
  "id": "srvtoolu_abc",
  "name": "code_execution",
  "input": {
    "code": "team = get_team_members('engineering')\n..." # the code example above
  }
}

**3. 工具执行而不影响 Claude 的上下文**

3. Tools execute without hitting Claude's context

当代码调用 `get_expenses()` 时，你会收到一个带有 `caller` 字段的工具请求：

When the code calls get_expenses(), you receive a tool request with a caller field:

```json
{
  "type": "tool_use",
  "id": "toolu_xyz",
  "name": "get_expenses",
  "input": {"user_id": "emp_123", "quarter": "Q3"},
  "caller": {
    "type": "code_execution_20250825",
    "tool_id": "srvtoolu_abc"
  }
}
```

{
  "type": "tool_use",
  "id": "toolu_xyz",
  "name": "get_expenses",
  "input": {"user_id": "emp_123", "quarter": "Q3"},
  "caller": {
    "type": "code_execution_20250825",
    "tool_id": "srvtoolu_abc"
  }
}

你提供结果，它在代码执行环境中处理，而不是在 Claude 的上下文中。这个请求-响应循环对代码中的每次工具调用都会重复。

You provide the result, which is processed in the Code Execution environment rather than Claude's context. This request-response cycle repeats for each tool call in the code.

**4. 只有最终输出进入上下文**

4. Only final output enters context

当代码完成运行时，只有代码的结果返回给 Claude：

When the code finishes running, only the results of the code are returned to Claude:

```json
{
  "type": "code_execution_tool_result",
  "tool_use_id": "srvtoolu_abc",
  "content": {
    "stdout": "[{\"name\": \"Alice\", \"spent\": 12500, \"limit\": 10000}...]"
  }
}
```

{
  "type": "code_execution_tool_result",
  "tool_use_id": "srvtoolu_abc",
  "content": {
    "stdout": "[{\"name\": \"Alice\", \"spent\": 12500, \"limit\": 10000}...]"
  }
}

这就是 Claude 看到的全部，而不是沿途处理的 2000+ 个费用明细项。

This is all Claude sees, not the 2000+ expense line items processed along the way.

### 何时使用编程式工具调用

When to use Programmatic Tool Calling

编程式工具调用在工作流中添加了代码执行步骤。当标记节省、延迟改进和准确性提升显著时，这种额外开销是值得的。

Programmatic Tool Calling adds a code execution step to your workflow. This extra overhead pays off when the token savings, latency improvements, and accuracy gains are substantial.

最有益的场景：

Most beneficial when:

处理大型数据集，你只需要聚合或摘要
运行具有三个或更多依赖工具调用的多步骤工作流
在 Claude 看到工具结果之前对其进行过滤、排序或转换
处理中间数据不应影响 Claude 推理的任务
对许多项目运行并行操作（例如，检查 50 个端点）

Processing large datasets where you only need aggregates or summaries
Running multi-step workflows with three or more dependent tool calls
Filtering, sorting, or transforming tool results before Claude sees them
Handling tasks where intermediate data shouldn't influence Claude's reasoning
Running parallel operations across many items (checking 50 endpoints, for example)

不太有益的场景：

Less beneficial when:

进行简单的单工具调用
处理 Claude 应该看到并推理所有中间结果的任务
使用小响应运行快速查找

Making simple single-tool invocations
Working on tasks where Claude should see and reason about all intermediate results
Running quick lookups with small responses

## 工具使用示例

Tool Use Examples

### 挑战

The challenge

JSON Schema 擅长定义结构——类型、必需字段、允许的枚举——但它无法表达使用模式：何时包含可选参数、哪些组合有意义，或你的 API 期望什么约定。

JSON Schema excels at defining structure–types, required fields, allowed enums–but it can't express usage patterns: when to include optional parameters, which combinations make sense, or what conventions your API expects.

考虑一个支持工单 API：

Consider a support ticket API:

```json
{
  "name": "create_ticket",
  "input_schema": {
    "properties": {
      "title": {"type": "string"},
      "priority": {"enum": ["low", "medium", "high", "critical"]},
      "labels": {"type": "array", "items": {"type": "string"}},
      "reporter": {
        "type": "object",
        "properties": {
          "id": {"type": "string"},
          "name": {"type": "string"},
          "contact": {
            "type": "object",
            "properties": {
              "email": {"type": "string"},
              "phone": {"type": "string"}
            }
          }
        }
      },
      "due_date": {"type": "string"},
      "escalation": {
        "type": "object",
        "properties": {
          "level": {"type": "integer"},
          "notify_manager": {"type": "boolean"},
          "sla_hours": {"type": "integer"}
        }
      }
    },
    "required": ["title"]
  }
}
```

{
  "name": "create_ticket",
  "input_schema": {
    "properties": {
      "title": {"type": "string"},
      "priority": {"enum": ["low", "medium", "high", "critical"]},
      "labels": {"type": "array", "items": {"type": "string"}},
      "reporter": {
        "type": "object",
        "properties": {
          "id": {"type": "string"},
          "name": {"type": "string"},
          "contact": {
            "type": "object",
            "properties": {
              "email": {"type": "string"},
              "phone": {"type": "string"}
            }
          }
        }
      },
      "due_date": {"type": "string"},
      "escalation": {
        "type": "object",
        "properties": {
          "level": {"type": "integer"},
          "notify_manager": {"type": "boolean"},
          "sla_hours": {"type": "integer"}
        }
      }
    },
    "required": ["title"]
  }
}

模式定义了什么是有效的，但留下了关键问题未解答：

The schema defines what's valid, but leaves critical questions unanswered:

**格式歧义**：`due_date` 应该使用"2024-11-06"、"Nov 6, 2024"还是"2024-11-06T00:00:00Z"？

Format ambiguity: Should due_date use "2024-11-06", "Nov 6, 2024", or "2024-11-06T00:00:00Z"?

**ID 约定**：`reporter.id` 是 UUID、"USR-12345"还是仅"12345"？

ID conventions: Is reporter.id a UUID, "USR-12345", or just "12345"?

**嵌套结构使用**：Claude 应该何时填充 `reporter.contact`？

Nested structure usage: When should Claude populate reporter.contact?

**参数相关性**：`escalation.level` 和 `escalation.sla_hours` 如何与 `priority` 相关？

Parameter correlations: How do escalation.level and escalation.sla_hours relate to priority?

这些歧义可能导致格式错误的工具调用和不一致的参数使用。

These ambiguities can lead to malformed tool calls and inconsistent parameter usage.

### 我们的解决方案

Our solution

工具使用示例让你直接在工具定义中提供示例工具调用。你不是仅依赖模式，而是向 Claude 展示具体的使用模式：

Tool Use Examples let you provide sample tool calls directly in your tool definitions. Instead of relying on schema alone, you show Claude concrete usage patterns:

```json
{
    "name": "create_ticket",
    "input_schema": { /* 与上面相同的模式 */ },
    "input_examples": [
      {
        "title": "登录页面返回 500 错误",
        "priority": "critical",
        "labels": ["bug", "authentication", "production"],
        "reporter": {
          "id": "USR-12345",
          "name": "Jane Smith",
          "contact": {
            "email": "jane@acme.com",
            "phone": "+1-555-0123"
          }
        },
        "due_date": "2024-11-06",
        "escalation": {
          "level": 2,
          "notify_manager": true,
          "sla_hours": 4
        }
      },
      {
        "title": "添加深色模式支持",
        "labels": ["feature-request", "ui"],
        "reporter": {
          "id": "USR-67890",
          "name": "Alex Chen"
        }
      },
      {
        "title": "更新 API 文档"
      }
    ]
  }
```

{
    "name": "create_ticket",
    "input_schema": { /* same schema as above */ },
    "input_examples": [
      {
        "title": "Login page returns 500 error",
        "priority": "critical",
        "labels": ["bug", "authentication", "production"],
        "reporter": {
          "id": "USR-12345",
          "name": "Jane Smith",
          "contact": {
            "email": "jane@acme.com",
            "phone": "+1-555-0123"
          }
        },
        "due_date": "2024-11-06",
        "escalation": {
          "level": 2,
          "notify_manager": true,
          "sla_hours": 4
        }
      },
      {
        "title": "Add dark mode support",
        "labels": ["feature-request", "ui"],
        "reporter": {
          "id": "USR-67890",
          "name": "Alex Chen"
        }
      },
      {
        "title": "Update API documentation"
      }
    ]
  }

从这三个示例中，Claude 学到：

From these three examples, Claude learns:

**格式约定**：日期使用 YYYY-MM-DD，用户 ID 遵循 USR-XXXXX，标签使用 kebab-case

Format conventions: Dates use YYYY-MM-DD, user IDs follow USR-XXXXX, labels use kebab-case

**嵌套结构模式**：如何构造带有嵌套 contact 对象的 reporter 对象

Nested structure patterns: How to construct the reporter object with its nested contact object

**可选参数相关性**：关键错误具有完整联系信息 + 升级和紧迫的 SLA；功能请求有 reporter 但没有 contact/escalation；内部任务只有标题

Optional parameter correlations: Critical bugs have full contact info + escalation with tight SLAs; feature requests have reporter but no contact/escalation; internal tasks have title only

在我们自己的内部测试中，工具使用示例将复杂参数处理的准确性从 72% 提高到 90%。

In our own internal testing, tool use examples improved accuracy from 72% to 90% on complex parameter handling.

### 何时使用工具使用示例

When to use Tool Use Examples

工具使用示例会向你的工具定义添加标记，因此当准确性改进超过额外成本时，它们最有价值。

Tool Use Examples add tokens to your tool definitions, so they're most valuable when accuracy improvements outweigh the additional cost.

最有益的场景：

Most beneficial when:

复杂的嵌套结构，其中有效的 JSON 并不意味着正确的使用
具有许多可选参数且包含模式很重要的工具
模式中未捕获的具有领域特定约定的 API
类似的工具，其中示例可以澄清使用哪个（例如，create_ticket 与 create_incident）

Complex nested structures where valid JSON doesn't imply correct usage
Tools with many optional parameters and inclusion patterns matter
APIs with domain-specific conventions not captured in schemas
Similar tools where examples clarify which one to use (e.g., create_ticket vs create_incident)

不太有益的场景：

Less beneficial when:

简单的单参数工具，使用显而易见
Claude 已经理解的标准格式，如 URL 或电子邮件
JSON Schema 约束更好处理的验证问题

Simple single-parameter tools with obvious usage
Standard formats like URLs or emails that Claude already understands
Validation concerns better handled by JSON Schema constraints

## 最佳实践

Best practices

构建采取实际行动的智能体意味着同时处理规模、复杂性和精确性。这三个功能协同工作，解决工具使用工作流中的不同瓶颈。以下是如何有效组合它们。

Building agents that take real-world actions means handling scale, complexity, and precision simultaneously. These three features work together to solve different bottlenecks in tool use workflows. Here's how to combine them effectively.

### 战略性地分层功能

Layer features strategically

并非每个智能体都需要为给定任务使用所有三个功能。从你最大的瓶颈开始：

Not every agent needs to use all three features for a given task. Start with your biggest bottleneck:

工具定义导致的上下文膨胀 → 工具搜索工具
污染上下文的大型中间结果 → 编程式工具调用
参数错误和格式错误的调用 → 工具使用示例

Context bloat from tool definitions → Tool Search Tool
Large intermediate results polluting context → Programmatic Tool Calling
Parameter errors and malformed calls → Tool Use Examples

这种聚焦方法让你解决限制智能体性能的特定约束，而不是预先增加复杂性。

This focused approach lets you address the specific constraint limiting your agent's performance, rather than adding complexity upfront.

然后根据需要分层添加其他功能。它们是互补的：工具搜索工具确保找到正确的工具，编程式工具调用确保高效执行，工具使用示例确保正确调用。

Then layer additional features as needed. They're complementary: Tool Search Tool ensures the right tools are found, Programmatic Tool Calling ensures efficient execution, and Tool Use Examples ensure correct invocation.

### 设置工具搜索工具以实现更好的发现

Set up Tool Search Tool for better discovery

工具搜索与名称和描述匹配，因此清晰、描述性的定义可以提高发现准确性。

Tool search matches against names and descriptions, so clear, descriptive definitions improve discovery accuracy.

```javascript
// 好
{
    "name": "search_customer_orders",
    "description": "按日期范围、状态或总金额搜索客户订单。返回订单详细信息，包括商品、运输和付款信息。"
}

// 差
{
    "name": "query_db_orders",
    "description": "执行订单查询"
}
```

// Good
{
    "name": "search_customer_orders",
    "description": "Search for customer orders by date range, status, or total amount. Returns order details including items, shipping, and payment info."
}

// Bad
{
    "name": "query_db_orders",
    "description": "Execute order query"
}

在系统提示中添加指导，以便 Claude 知道可用的内容：

Add system prompt guidance so Claude knows what's available:

```
你可以访问用于 Slack 消息传递、Google Drive 文件管理、
Jira 工单跟踪和 GitHub 仓库操作的工具。使用工具搜索
查找特定功能。
```

You have access to tools for Slack messaging, Google Drive file management,
Jira ticket tracking, and GitHub repository operations. Use the tool search
to find specific capabilities.

保持你最常用的三到五个工具始终加载，延迟加载其余工具。这在常见操作的即时访问和其他所有内容的按需发现之间取得平衡。

Keep your three to five most-used tools always loaded, defer the rest. This balances immediate access for common operations with on-demand discovery for everything else.

### 设置编程式工具调用以实现正确执行

Set up Programmatic Tool Calling for correct execution

由于 Claude 编写代码来解析工具输出，请清楚地记录返回格式。这有助于 Claude 编写正确的解析逻辑：

Since Claude writes code to parse tool outputs, document return formats clearly. This helps Claude write correct parsing logic:

```json
{
    "name": "get_orders",
    "description": "检索客户的订单。
返回：
    订单对象列表，每个包含：
    - id (str)：订单标识符
    - total (float)：订单总额（美元）
    - status (str)：'pending'、'shipped'、'delivered' 之一
    - items (list)：{sku, quantity, price} 数组
    - created_at (str)：ISO 8601 时间戳"
}
```

{
    "name": "get_orders",
    "description": "Retrieve orders for a customer.
Returns:
    List of order objects, each containing:
    - id (str): Order identifier
    - total (float): Order total in USD
    - status (str): One of 'pending', 'shipped', 'delivered'
    - items (list): Array of {sku, quantity, price}
    - created_at (str): ISO 8601 timestamp"
}

查看以下选择性启用从编程编排中受益的工具：

See below for opt-in tools that benefit from programmatic orchestration:

可以并行运行的工具（独立操作）
安全重试的操作（幂等）

Tools that can run in parallel (independent operations)
Operations safe to retry (idempotent)

### 设置工具使用示例以提高参数准确性

Set up Tool Use Examples for parameter accuracy

制作示例以实现行为清晰性：

Craft examples for behavioral clarity:

使用真实数据（真实城市名称、合理价格，而不是"字符串"或"值"）
用最小、部分和完整规范模式展示多样性
保持简洁：每个工具 1-5 个示例
关注歧义（只在正确使用从模式中不明显时添加示例）

Use realistic data (real city names, plausible prices, not "string" or "value")
Show variety with minimal, partial, and full specification patterns
Keep it concise: 1-5 examples per tool
Focus on ambiguity (only add examples where correct usage isn't obvious from schema)

## 开始使用

Getting started

这些功能在测试版中可用。要启用它们，请添加 beta 标头并包含你需要的工具：

These features are available in beta. To enable them, add the beta header and include the tools you need:

```python
client.beta.messages.create(
    betas=["advanced-tool-use-2025-11-20"],
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    tools=[
        {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},
        {"type": "code_execution_20250825", "name": "code_execution"},
        # 你的工具，带有 defer_loading、allowed_callers 和 input_examples
    ]
)
```

client.beta.messages.create(
    betas=["advanced-tool-use-2025-11-20"],
    model="claude-sonnet-4-5-20250929",
    max_tokens=4096,
    tools=[
        {"type": "tool_search_tool_regex_20251119", "name": "tool_search_tool_regex"},
        {"type": "code_execution_20250825", "name": "code_execution"},
        # Your tools with defer_loading, allowed_callers, and input_examples
    ]
)

有关详细的 API 文档和 SDK 示例，请参阅我们的：

For detailed API documentation and SDK examples, see our:

工具搜索工具的文档和 cookbook
编程式工具调用的文档和 cookbook
工具使用示例的文档

Documentation and cookbook for Tool Search Tool
Documentation and cookbook for Programmatic Tool Calling
Documentation for Tool Use Examples

这些功能将工具使用从简单的函数调用转向智能编排。随着智能体处理跨越数十个工具和大型数据集的更复杂工作流，动态发现、高效执行和可靠调用变得至关重要。

These features move tool use from simple function calling toward intelligent orchestration. As agents tackle more complex workflows spanning dozens of tools and large datasets, dynamic discovery, efficient execution, and reliable invocation become foundational.

我们很期待看到你会构建什么。

We're excited to see what you build.

## 致谢

Acknowledgements

本文由 Bin Wu 撰写，Adam Jones、Artur Renault、Henry Tay、Jake Noble、Nathan McCandlish、Noah Picard、Sam Jiang 和 Claude 开发者平台团队做出了贡献。这项工作建立在 Chris Gorgolewski、Daniel Jiang、Jeremy Fox 和 Mike Lambert 的基础研究之上。我们还从整个 AI 生态系统中汲取灵感，包括 Joel Pobar 的 LLMVM、Cloudflare 的 Code Mode 和 Code Execution as MCP。特别感谢 Andy Schumeister、Hamish Kerr、Keir Bradwell、Matt Bleifer 和 Molly Vorwerck 的支持。

Written by Bin Wu, with contributions from Adam Jones, Artur Renault, Henry Tay, Jake Noble, Nathan McCandlish, Noah Picard, Sam Jiang, and the Claude Developer Platform team. This work builds on foundational research by Chris Gorgolewski, Daniel Jiang, Jeremy Fox and Mike Lambert. We also drew inspiration from across the AI ecosystem, including Joel Pobar's LLMVM, Cloudflare's Code Mode and Code Execution as MCP. Special thanks to Andy Schumeister, Hamish Kerr, Keir Bradwell, Matt Bleifer and Molly Vorwerck for their support.
