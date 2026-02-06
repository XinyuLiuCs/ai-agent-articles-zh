# 为智能体编写高效工具——与智能体协作

*Writing effective tools for agents — with agents*---
https://www.anthropic.com/engineering/writing-tools-for-agents

发布日期：2025年9月11日

Published Sep 11, 2025

智能体的效能取决于我们赋予它们的工具。我们分享如何编写高质量的工具和评估，以及如何利用 Claude 来优化它自身的工具以提升性能。

Agents are only as effective as the tools we give them. We share how to write high-quality tools and evaluations, and how you can boost performance by using Claude to optimize its tools for itself.

模型上下文协议（MCP）可以赋予 LLM 智能体数百种工具来解决现实世界的任务。但我们如何让这些工具发挥最大效用？

The Model Context Protocol (MCP) can empower LLM agents with potentially hundreds of tools to solve real-world tasks. But how do we make those tools maximally effective?

在这篇文章中，我们描述了在各种智能体 AI 系统中提升性能的最有效技术<sup>1</sup>。

In this post, we describe our most effective techniques for improving performance in a variety of agentic AI systems<sup>1</sup>.

我们首先介绍如何：

We begin by covering how you can:

- 构建和测试工具原型
- 创建和运行工具与智能体的综合评估
- 与 Claude Code 等智能体协作，自动提升工具性能

- Build and test prototypes of your tools
- Create and run comprehensive evaluations of your tools with agents
- Collaborate with agents like Claude Code to automatically increase the performance of your tools

最后，我们总结了在此过程中发现的编写高质量工具的关键原则：

We conclude with key principles for writing high-quality tools we've identified along the way:

- 选择正确的工具来实现（以及不实现哪些）
- 使用命名空间划分工具的功能边界
- 从工具向智能体返回有意义的上下文
- 优化工具响应的 token 效率
- 对工具描述和规格进行提示工程

- Choosing the right tools to implement (and not to implement)
- Namespacing tools to define clear boundaries in functionality
- Returning meaningful context from tools back to agents
- Optimizing tool responses for token efficiency
- Prompt-engineering tool descriptions and specs

> **图示：工程师如何使用 Claude Code 评估智能体工具的效能**
>
> 构建评估体系使你能够系统性地衡量工具的性能。你可以使用 Claude Code 针对该评估自动优化你的工具。

> **Illustration: How an engineer might use Claude Code to evaluate the efficacy of agentic tools**
>
> Building an evaluation allows you to systematically measure the performance of your tools. You can use Claude Code to automatically optimize your tools against this evaluation.

## 什么是工具？

What is a tool?

在计算领域，确定性系统在给定相同输入时每次都会产生相同的输出，而非确定性系统——如智能体——即使在相同的起始条件下也可能生成不同的响应。

In computing, deterministic systems produce the same output every time given identical inputs, while non-deterministic systems—like agents—can generate varied responses even with the same starting conditions.

当我们传统地编写软件时，我们在确定性系统之间建立契约。例如，像 `getWeather("NYC")` 这样的函数调用每次被调用时都会以完全相同的方式获取纽约市的天气。

When we traditionally write software, we're establishing a contract between deterministic systems. For instance, a function call like `getWeather("NYC")` will always fetch the weather in New York City in the exact same manner every time it is called.

工具是一种新型软件，它反映了确定性系统与非确定性智能体之间的契约。当用户问"我今天需要带伞吗？"时，智能体可能会调用天气工具、根据通用知识回答，甚至先询问一个关于地点的澄清问题。有时，智能体可能会产生幻觉，甚至无法理解如何使用工具。

Tools are a new kind of software which reflects a contract between deterministic systems and non-deterministic agents. When a user asks "Should I bring an umbrella today?," an agent might call the weather tool, answer from general knowledge, or even ask a clarifying question about location first. Occasionally, an agent might hallucinate or even fail to grasp how to use a tool.

这意味着在为智能体编写软件时需要从根本上重新思考方法：我们不能像为其他开发者或系统编写函数和 API 那样编写工具和 MCP 服务器，而需要针对智能体来设计它们。

This means fundamentally rethinking our approach when writing software for agents: instead of writing tools and MCP servers the way we'd write functions and APIs for other developers or systems, we need to design them for agents.

我们的目标是扩大智能体能够有效利用工具解决广泛任务的范围，让它们能追求多种成功策略。幸运的是，根据我们的经验，那些对智能体最"符合人体工学"的工具，对人类来说也出奇地直观易懂。

Our goal is to increase the surface area over which agents can be effective in solving a wide range of tasks by using tools to pursue a variety of successful strategies. Fortunately, in our experience, the tools that are most "ergonomic" for agents also end up being surprisingly intuitive to grasp as humans.

## 如何编写工具

How to write tools

在本节中，我们描述如何与智能体协作来编写和改进你赋予它们的工具。首先快速搭建工具原型并在本地测试。接下来，运行综合评估来衡量后续变更的效果。与智能体协作，你可以反复进行评估和改进，直到你的智能体在真实世界任务上达到强劲表现。

In this section, we describe how you can collaborate with agents both to write and to improve the tools you give them. Start by standing up a quick prototype of your tools and testing them locally. Next, run a comprehensive evaluation to measure subsequent changes. Working alongside agents, you can repeat the process of evaluating and improving your tools until your agents achieve strong performance on real-world tasks.

### 构建原型

Building a prototype

在实际动手之前，很难预判哪些工具对智能体来说是好用的，哪些不是。先快速搭建一个工具原型。如果你使用 Claude Code 来编写工具（可能是一次性生成），为 Claude 提供工具将依赖的软件库、API 或 SDK（包括可能的 MCP SDK）的文档会很有帮助。LLM 友好的文档通常可以在官方文档网站的扁平 `llms.txt` 文件中找到（[这是我们 API 的](https://docs.anthropic.com/llms.txt)）。

It can be difficult to anticipate which tools agents will find ergonomic and which tools they won't without getting hands-on yourself. Start by standing up a quick prototype of your tools. If you're using Claude Code to write your tools (potentially in one-shot), it helps to give Claude documentation for any software libraries, APIs, or SDKs (including potentially the MCP SDK) your tools will rely on. LLM-friendly documentation can commonly be found in flat `llms.txt` files on official documentation sites ([here's our API's](https://docs.anthropic.com/llms.txt)).

将工具封装在本地 MCP 服务器或桌面扩展（DXT）中，可以让你在 Claude Code 或 Claude 桌面应用中连接和测试工具。

Wrapping your tools in a local MCP server or Desktop extension (DXT) will allow you to connect and test your tools in Claude Code or the Claude Desktop app.

要将本地 MCP 服务器连接到 Claude Code，运行 `claude mcp add <name> <command> [args...]`。

To connect your local MCP server to Claude Code, run `claude mcp add <name> <command> [args...]`.

要将本地 MCP 服务器或 DXT 连接到 Claude 桌面应用，分别导航到"设置 > 开发者"或"设置 > 扩展"。

To connect your local MCP server or DXT to the Claude Desktop app, navigate to Settings > Developer or Settings > Extensions, respectively.

工具也可以直接传入 Anthropic API 调用以进行编程测试。

Tools can also be passed directly into Anthropic API calls for programmatic testing.

亲自测试工具以发现任何粗糙之处。收集用户反馈，对你期望工具支持的使用场景和提示建立直觉。

Test the tools yourself to identify any rough edges. Collect feedback from your users to build an intuition around the use-cases and prompts you expect your tools to enable.

### 运行评估

Running an evaluation

接下来，你需要通过运行评估来衡量 Claude 使用工具的效果。首先生成大量基于真实世界用例的评估任务。我们建议与智能体协作来帮助分析结果并确定如何改进工具。请在我们的[工具评估 cookbook](https://github.com/anthropics/anthropic-cookbook) 中查看这一端到端流程。

Next, you need to measure how well Claude uses your tools by running an evaluation. Start by generating lots of evaluation tasks, grounded in real world uses. We recommend collaborating with an agent to help analyze your results and determine how to improve your tools. See this process end-to-end in our tool evaluation cookbook.

> **图表：人工编写与 Claude 优化的 Slack MCP 服务器的测试集准确率**
>
> 我们内部 Slack 工具在留出测试集上的表现。

> **Chart: Test set accuracy of human-written vs. Claude-optimized Slack MCP servers**
>
> Held-out test set performance of our internal Slack tools

#### 生成评估任务

Generating evaluation tasks

利用早期原型，Claude Code 可以快速探索你的工具并创建数十个提示与响应对。提示应受真实世界用例启发，并基于实际的数据源和服务（例如内部知识库和微服务）。我们建议避免过于简单或肤浅的"沙箱"环境——这些环境无法以足够的复杂度对工具进行压力测试。强有力的评估任务可能需要多次工具调用——甚至数十次。

With your early prototype, Claude Code can quickly explore your tools and create dozens of prompt and response pairs. Prompts should be inspired by real-world uses and be based on realistic data sources and services (for example, internal knowledge bases and microservices). We recommend you avoid overly simplistic or superficial "sandbox" environments that don't stress-test your tools with sufficient complexity. Strong evaluation tasks might require multiple tool calls—potentially dozens.

以下是一些强评估任务的示例：

Here are some examples of strong tasks:

- 安排下周与 Jane 开会讨论我们最新的 Acme Corp 项目。附上上次项目规划会议的笔记并预订一间会议室。
- 客户 ID 9182 报告称单次购买尝试被扣了三次款。查找所有相关日志条目，并确定是否有其他客户受到同一问题的影响。
- 客户 Sarah Chen 刚提交了一份取消请求。准备一份挽留方案。确定：(1) 他们离开的原因，(2) 什么挽留方案最有说服力，(3) 在提出方案之前我们应注意的任何风险因素。

- Schedule a meeting with Jane next week to discuss our latest Acme Corp project. Attach the notes from our last project planning meeting and reserve a conference room.
- Customer ID 9182 reported that they were charged three times for a single purchase attempt. Find all relevant log entries and determine if any other customers were affected by the same issue.
- Customer Sarah Chen just submitted a cancellation request. Prepare a retention offer. Determine: (1) why they're leaving, (2) what retention offer would be most compelling, and (3) any risk factors we should be aware of before making an offer.

以下是一些较弱任务的示例：

And here are some weaker tasks:

- 安排下周与 jane@acme.corp 开会。
- 在支付日志中搜索 purchase_complete 和 customer_id=9182。
- 查找客户 ID 45892 的取消请求。

- Schedule a meeting with jane@acme.corp next week.
- Search the payment logs for purchase_complete and customer_id=9182.
- Find the cancellation request by Customer ID 45892.

每个评估提示应与一个可验证的响应或结果配对。你的验证器可以简单到将真实答案与采样响应进行精确字符串比较，也可以高级到使用 Claude 来评判响应。避免使用过于严格的验证器——它们可能因格式、标点或有效的替代措辞等无关差异而拒绝正确响应。

Each evaluation prompt should be paired with a verifiable response or outcome. Your verifier can be as simple as an exact string comparison between ground truth and sampled responses, or as advanced as enlisting Claude to judge the response. Avoid overly strict verifiers that reject correct responses due to spurious differences like formatting, punctuation, or valid alternative phrasings.

对于每个提示-响应对，你还可以选择性地指定预期智能体在解决任务时调用的工具，以衡量智能体在评估过程中是否成功理解了每个工具的用途。不过，由于正确解决任务可能有多条有效路径，请尽量避免过度指定或过拟合于特定策略。

For each prompt-response pair, you can optionally also specify the tools you expect an agent to call in solving the task, to measure whether or not agents are successful in grasping each tool's purpose during evaluation. However, because there might be multiple valid paths to solving tasks correctly, try to avoid overspecifying or overfitting to strategies.

#### 运行评估

Running the evaluation

我们建议使用直接的 LLM API 调用以编程方式运行评估。使用简单的智能体循环（交替调用 LLM API 和工具的 while 循环）：每个评估任务一个循环。每个评估智能体应获得单个任务提示和你的工具。

We recommend running your evaluation programmatically with direct LLM API calls. Use simple agentic loops (while-loops wrapping alternating LLM API and tool calls): one loop for each evaluation task. Each evaluation agent should be given a single task prompt and your tools.

在评估智能体的系统提示中，我们建议指示智能体不仅输出结构化的响应块（用于验证），还要输出推理和反馈块。指示智能体在工具调用和响应块之前输出这些内容，可能通过触发思维链（CoT）行为来提升 LLM 的有效智能。

In your evaluation agents' system prompts, we recommend instructing agents to output not just structured response blocks (for verification), but also reasoning and feedback blocks. Instructing agents to output these before tool call and response blocks may increase LLMs' effective intelligence by triggering chain-of-thought (CoT) behaviors.

如果你使用 Claude 运行评估，可以开启交错思考（interleaved thinking）以获得类似的"开箱即用"功能。这将帮助你探究智能体为什么调用或不调用某些工具，并突出工具描述和规格中需要改进的具体领域。

If you're running your evaluation with Claude, you can turn on interleaved thinking for similar functionality "off-the-shelf". This will help you probe why agents do or don't call certain tools and highlight specific areas of improvement in tool descriptions and specs.

除了整体准确率外，我们还建议收集其他指标，如单个工具调用和任务的总运行时间、工具调用总数、总 token 消耗和工具错误。跟踪工具调用有助于揭示智能体追求的常见工作流，并为工具整合提供机会。

As well as top-level accuracy, we recommend collecting other metrics like the total runtime of individual tool calls and tasks, the total number of tool calls, the total token consumption, and tool errors. Tracking tool calls can help reveal common workflows that agents pursue and offer some opportunities for tools to consolidate.

> **图表：人工编写与 Claude 优化的 Asana MCP 服务器的测试集准确率**
>
> 我们内部 Asana 工具在留出测试集上的表现。

> **Chart: Test set accuracy of human-written vs. Claude-optimized Asana MCP servers**
>
> Held-out test set performance of our internal Asana tools

#### 分析结果

Analyzing results

智能体是你在发现问题和提供反馈方面的得力伙伴，从相互矛盾的工具描述到低效的工具实现和令人困惑的工具 schema，无所不包。然而，请记住，智能体在反馈和响应中遗漏的内容往往比包含的内容更重要。LLM 并不总是言出必行。

Agents are your helpful partners in spotting issues and providing feedback on everything from contradictory tool descriptions to inefficient tool implementations and confusing tool schemas. However, keep in mind that what agents omit in their feedback and responses can often be more important than what they include. LLMs don't always say what they mean.

观察你的智能体在哪里卡住或困惑。通读评估智能体的推理和反馈（或 CoT）以识别粗糙之处。审查原始转录内容（包括工具调用和工具响应）以捕获智能体 CoT 中未明确描述的行为。学会读懂字里行间的含义——记住，你的评估智能体不一定知道正确答案和策略。

Observe where your agents get stumped or confused. Read through your evaluation agents' reasoning and feedback (or CoT) to identify rough edges. Review the raw transcripts (including tool calls and tool responses) to catch any behavior not explicitly described in the agent's CoT. Read between the lines; remember that your evaluation agents don't necessarily know the correct answers and strategies.

分析你的工具调用指标。大量冗余的工具调用可能表明需要调整分页或 token 限制参数；大量因无效参数导致的工具错误可能表明工具需要更清晰的描述或更好的示例。当我们推出 Claude 的网页搜索工具时，我们发现 Claude 在工具的查询参数后不必要地附加了 2025，导致搜索结果产生偏差并降低性能（我们通过改进工具描述引导 Claude 走上了正确的方向）。

Analyze your tool calling metrics. Lots of redundant tool calls might suggest some rightsizing of pagination or token limit parameters is warranted; lots of tool errors for invalid parameters might suggest tools could use clearer descriptions or better examples. When we launched Claude's web search tool, we identified that Claude was needlessly appending 2025 to the tool's query parameter, biasing search results and degrading performance (we steered Claude in the right direction by improving the tool description).

#### 与智能体协作

Collaborating with agents

你甚至可以让智能体分析你的结果并为你改进工具。只需将评估智能体的转录内容连接起来并粘贴到 Claude Code 中。Claude 擅长分析转录内容并一次性重构大量工具——例如，确保在进行新更改时工具实现和描述保持自洽。

You can even let agents analyze your results and improve your tools for you. Simply concatenate the transcripts from your evaluation agents and paste them into Claude Code. Claude is an expert at analyzing transcripts and refactoring lots of tools all at once—for example, to ensure tool implementations and descriptions remain self-consistent when new changes are made.

事实上，这篇文章中的大部分建议都来自于使用 Claude Code 反复优化我们内部工具实现的过程。我们的评估是基于内部工作区构建的，反映了内部工作流的复杂性，包括真实的项目、文档和消息。

In fact, most of the advice in this post came from repeatedly optimizing our internal tool implementations with Claude Code. Our evaluations were created on top of our internal workspace, mirroring the complexity of our internal workflows, including real projects, documents, and messages.

我们依赖留出测试集来确保不会过拟合于"训练"评估。这些测试集表明，即使超出"专家"工具实现所达到的水平，我们仍能提取额外的性能提升——无论这些工具是由我们的研究人员手动编写的还是由 Claude 本身生成的。

We relied on held-out test sets to ensure we did not overfit to our "training" evaluations. These test sets revealed that we could extract additional performance improvements even beyond what we achieved with "expert" tool implementations—whether those tools were manually written by our researchers or generated by Claude itself.

在下一节中，我们将分享从这一过程中学到的一些经验。

In the next section, we'll share some of what we learned from this process.

## 编写高效工具的原则

Principles for writing effective tools

在本节中，我们将学到的经验提炼为几个编写高效工具的指导原则。

In this section, we distill our learnings into a few guiding principles for writing effective tools.

### 为智能体选择正确的工具

Choosing the right tools for agents

更多的工具并不总是带来更好的结果。我们观察到的一个常见错误是，工具仅仅包装了现有的软件功能或 API 端点——而不考虑这些工具是否适合智能体使用。这是因为智能体与传统软件有着不同的"可供性"（affordances）——也就是说，它们感知和利用工具采取行动的方式不同。

More tools don't always lead to better outcomes. A common error we've observed is tools that merely wrap existing software functionality or API endpoints—whether or not the tools are appropriate for agents. This is because agents have distinct "affordances" to traditional software—that is, they have different ways of perceiving the potential actions they can take with those tools.

LLM 智能体的"上下文"是有限的（也就是说，它们一次能处理的信息量有上限），而计算机内存则廉价且充足。以在通讯录中搜索联系人为例。传统软件程序可以高效地逐一存储和处理联系人列表，逐个检查后再继续。

LLM agents have limited "context" (that is, there are limits to how much information they can process at once), whereas computer memory is cheap and abundant. Consider the task of searching for a contact in an address book. Traditional software programs can efficiently store and process a list of contacts one at a time, checking each one before moving on.

然而，如果 LLM 智能体使用一个返回所有联系人的工具，然后不得不逐 token 读取每一条，它就是在浪费有限的上下文空间处理无关信息（想象一下通过从头到尾逐页阅读来搜索通讯录中的联系人——即暴力搜索）。更好、更自然的方法（对智能体和人类都是如此）是先跳到相关页面（也许按字母顺序查找）。

However, if an LLM agent uses a tool that returns ALL contacts and then has to read through each one token-by-token, it's wasting its limited context space on irrelevant information (imagine searching for a contact in your address book by reading each page from top-to-bottom—that is, via brute-force search). The better and more natural approach (for agents and humans alike) is to skip to the relevant page first (perhaps finding it alphabetically).

我们建议构建少量深思熟虑的工具，针对特定的高影响力工作流，与你的评估任务匹配，然后从那里扩展。在通讯录的例子中，你可能会选择实现 `search_contacts` 或 `message_contact` 工具，而不是 `list_contacts` 工具。

We recommend building a few thoughtful tools targeting specific high-impact workflows, which match your evaluation tasks and scaling up from there. In the address book case, you might choose to implement a `search_contacts` or `message_contact` tool instead of a `list_contacts` tool.

工具可以整合功能，在底层处理多个离散操作（或 API 调用）。例如，工具可以用相关元数据丰富工具响应，或在单次工具调用中处理经常串联的多步骤任务。

Tools can consolidate functionality, handling potentially multiple discrete operations (or API calls) under the hood. For example, tools can enrich tool responses with related metadata or handle frequently chained, multi-step tasks in a single tool call.

以下是一些示例：

Here are some examples:

- 与其实现 `list_users`、`list_events` 和 `create_event` 工具，不如考虑实现一个 `schedule_event` 工具来查找可用时间并安排事件。
- 与其实现 `read_logs` 工具，不如考虑实现一个 `search_logs` 工具，只返回相关日志行和一些周围上下文。
- 与其实现 `get_customer_by_id`、`list_transactions` 和 `list_notes` 工具，不如实现一个 `get_customer_context` 工具，一次性汇总客户所有近期和相关信息。

- Instead of implementing a `list_users`, `list_events`, and `create_event` tools, consider implementing a `schedule_event` tool which finds availability and schedules an event.
- Instead of implementing a `read_logs` tool, consider implementing a `search_logs` tool which only returns relevant log lines and some surrounding context.
- Instead of implementing `get_customer_by_id`, `list_transactions`, and `list_notes` tools, implement a `get_customer_context` tool which compiles all of a customer's recent & relevant information all at once.

确保你构建的每个工具都有明确、独特的用途。工具应使智能体能够以人类在获得相同底层资源时的方式来分解和解决任务，同时减少中间输出本来会消耗的上下文。

Make sure each tool you build has a clear, distinct purpose. Tools should enable agents to subdivide and solve tasks in much the same way that a human would, given access to the same underlying resources, and simultaneously reduce the context that would have otherwise been consumed by intermediate outputs.

过多的工具或功能重叠的工具也会分散智能体追求高效策略的注意力。仔细、有选择性地规划你要构建（或不构建）的工具确实能带来显著回报。

Too many tools or overlapping tools can also distract agents from pursuing efficient strategies. Careful, selective planning of the tools you build (or don't build) can really pay off.

### 为工具添加命名空间

Namespacing your tools

你的 AI 智能体可能会访问数十个 MCP 服务器和数百种不同的工具——包括其他开发者提供的工具。当工具在功能上重叠或用途模糊时，智能体可能会搞不清该使用哪一个。

Your AI agents will potentially gain access to dozens of MCP servers and hundreds of different tools–including those by other developers. When tools overlap in function or have a vague purpose, agents can get confused about which ones to use.

命名空间（将相关工具归入共同前缀下）有助于在大量工具之间划定边界；MCP 客户端有时会默认这样做。例如，按服务命名空间（如 `asana_search`、`jira_search`）和按资源命名空间（如 `asana_projects_search`、`asana_users_search`），可以帮助智能体在正确的时机选择正确的工具。

Namespacing (grouping related tools under common prefixes) can help delineate boundaries between lots of tools; MCP clients sometimes do this by default. For example, namespacing tools by service (e.g., `asana_search`, `jira_search`) and by resource (e.g., `asana_projects_search`, `asana_users_search`), can help agents select the right tools at the right time.

我们发现，在前缀式和后缀式命名空间之间的选择对工具使用评估有不可忽视的影响。效果因 LLM 而异，我们鼓励你根据自己的评估来选择命名方案。

We have found selecting between prefix- and suffix-based namespacing to have non-trivial effects on our tool-use evaluations. Effects vary by LLM and we encourage you to choose a naming scheme according to your own evaluations.

智能体可能会调用错误的工具、用错误的参数调用正确的工具、调用的工具太少，或者错误地处理工具响应。通过有选择性地实现名称反映任务自然分解方式的工具，你可以同时减少加载到智能体上下文中的工具数量和工具描述，并将智能体计算从智能体上下文卸载回工具调用本身。这降低了智能体犯错的总体风险。

Agents might call the wrong tools, call the right tools with the wrong parameters, call too few tools, or process tool responses incorrectly. By selectively implementing tools whose names reflect natural subdivisions of tasks, you simultaneously reduce the number of tools and tool descriptions loaded into the agent's context and offload agentic computation from the agent's context back into the tool calls themselves. This reduces an agent's overall risk of making mistakes.

### 从工具返回有意义的上下文

Returning meaningful context from your tools

同样，工具实现应注意只向智能体返回高信号量的信息。它们应优先考虑上下文相关性而非灵活性，避免低级技术标识符（例如：`uuid`、`256px_image_url`、`mime_type`）。诸如 `name`、`image_url` 和 `file_type` 这样的字段更有可能直接影响智能体的后续行动和响应。

In the same vein, tool implementations should take care to return only high signal information back to agents. They should prioritize contextual relevance over flexibility, and eschew low-level technical identifiers (for example: `uuid`, `256px_image_url`, `mime_type`). Fields like `name`, `image_url`, and `file_type` are much more likely to directly inform agents' downstream actions and responses.

**智能体处理自然语言名称、术语或标识符的能力也明显优于处理晦涩标识符的能力。我们发现，仅仅将任意的字母数字 UUID 解析为更具语义意义和可解释性的语言（甚至是从 0 开始的 ID 方案），就能通过减少幻觉显著提高 Claude 在检索任务中的精确度。**

Agents also tend to grapple with natural language names, terms, or identifiers significantly more successfully than they do with cryptic identifiers. We've found that merely resolving arbitrary alphanumeric UUIDs to more semantically meaningful and interpretable language (or even a 0-indexed ID scheme) significantly improves Claude's precision in retrieval tasks by reducing hallucinations.

在某些情况下，智能体可能需要灵活地同时使用自然语言和技术标识符输出，即使只是为了触发下游工具调用（例如，`search_user(name='jane')` → `send_message(id=12345)`）。你可以通过在工具中暴露一个简单的 `response_format` 枚举参数来实现这两种需求，让智能体控制工具返回"简洁"还是"详细"的响应（见下图）。

In some instances, agents may require the flexibility to interact with both natural language and technical identifiers outputs, if only to trigger downstream tool calls (for example, `search_user(name='jane')` → `send_message(id=12345)`). You can enable both by exposing a simple `response_format` enum parameter in your tool, allowing your agent to control whether tools return "concise" or "detailed" responses (images below).

你可以添加更多格式以获得更大的灵活性，类似于 GraphQL 中可以精确选择想要接收的信息片段。以下是一个用于控制工具响应详细程度的 `ResponseFormat` 枚举示例：

You can add more formats for even greater flexibility, similar to GraphQL where you can choose exactly which pieces of information you want to receive. Here is an example `ResponseFormat` enum to control tool response verbosity:

```
enum ResponseFormat {
   DETAILED = "detailed",
   CONCISE = "concise"
}
```

> **详细工具响应示例（206 tokens）**
>
> Slack 对话线程和回复通过唯一的 `thread_ts` 标识，获取线程回复时需要该标识。`thread_ts` 和其他 ID（`channel_id`、`user_id`）可从"详细"工具响应中获取，以支持需要这些信息的后续工具调用。"简洁"工具响应仅返回对话内容，不包含 ID。在此示例中，使用"简洁"响应大约只消耗 1/3 的 token。

> **Example of a detailed tool response (206 tokens)**
>
> Slack threads and thread replies are identified by unique `thread_ts` which are required to fetch thread replies. `thread_ts` and other IDs (`channel_id`, `user_id`) can be retrieved from a "detailed" tool response to enable further tool calls that require these. "concise" tool responses return only thread content and exclude IDs. In this example, we use ~⅓ of the tokens with "concise" tool responses.

> **简洁工具响应示例（72 tokens）**

> **Example of a concise tool response (72 tokens)**

即使工具响应的结构——例如 XML、JSON 或 Markdown——也可能对评估性能产生影响：没有放之四海而皆准的方案。这是因为 LLM 基于下一 token 预测训练，往往在与训练数据匹配的格式上表现更好。最优的响应结构因任务和智能体而异。我们鼓励你根据自己的评估来选择最佳的响应结构。

Even your tool response structure—for example XML, JSON, or Markdown—can have an impact on evaluation performance: there is no one-size-fits-all solution. This is because LLMs are trained on next-token prediction and tend to perform better with formats that match their training data. The optimal response structure will vary widely by task and agent. We encourage you to select the best response structure based on your own evaluation.

### 优化工具响应的 token 效率

Optimizing tool responses for token efficiency

优化上下文质量很重要。但优化工具响应中返回给智能体的上下文数量同样重要。

Optimizing the quality of context is important. But so is optimizing the quantity of context returned back to agents in tool responses.

我们建议对任何可能消耗大量上下文的工具响应实现分页、范围选择、过滤和/或截断的某种组合，并设置合理的默认参数值。**对于 Claude Code，我们默认将工具响应限制为 25,000 tokens。**我们预期智能体的有效上下文长度将随时间增长，但对上下文高效工具的需求将持续存在。

We suggest implementing some combination of pagination, range selection, filtering, and/or truncation with sensible default parameter values for any tool responses that could use up lots of context. For Claude Code, we restrict tool responses to 25,000 tokens by default. We expect the effective context length of agents to grow over time, but the need for context-efficient tools to remain.

如果你选择截断响应，务必用有用的指引来引导智能体。你可以直接鼓励智能体追求更高 token 效率的策略，例如在知识检索任务中进行多次小而有针对性的搜索，而非单次宽泛搜索。同样，如果工具调用引发错误（例如输入验证失败），你可以对错误响应进行提示工程，清晰传达具体且可操作的改进建议，而非返回晦涩的错误代码或堆栈跟踪。

If you choose to truncate responses, be sure to steer agents with helpful instructions. You can directly encourage agents to pursue more token-efficient strategies, like making many small and targeted searches instead of a single, broad search for a knowledge retrieval task. Similarly, if a tool call raises an error (for example, during input validation), you can prompt-engineer your error responses to clearly communicate specific and actionable improvements, rather than opaque error codes or tracebacks.

> **截断工具响应示例**
>
> **不佳的错误响应示例**
>
> **良好的错误响应示例**
>
> 工具截断和错误响应可以引导智能体采用更高 token 效率的工具使用行为（使用过滤器或分页），或提供正确格式化工具输入的示例。

> **Example of a truncated tool response**
>
> **Example of an unhelpful error response**
>
> **Example of a helpful error response**
>
> Tool truncation and error responses can steer agents towards more token-efficient tool-use behaviors (using filters or pagination) or give examples of correctly formatted tool inputs.

### 对工具描述进行提示工程

Prompt-engineering your tool descriptions

现在我们来到改进工具最有效的方法之一：对工具描述和规格进行提示工程。由于这些描述会被加载到智能体的上下文中，它们可以共同引导智能体采取有效的工具调用行为。

We now come to one of the most effective methods for improving tools: prompt-engineering your tool descriptions and specs. Because these are loaded into your agents' context, they can collectively steer agents toward effective tool-calling behaviors.

编写工具描述和规格时，想象你如何向团队中的新成员描述你的工具。考虑你可能隐含携带的上下文——专门的查询格式、小众术语的定义、底层资源之间的关系——并将其显式化。通过清晰描述（并用严格的数据模型强制执行）预期的输入和输出来避免歧义。特别是，输入参数应命名明确：不要使用名为 `user` 的参数，而应使用 `user_id`。

When writing tool descriptions and specs, think of how you would describe your tool to a new hire on your team. Consider the context that you might implicitly bring—specialized query formats, definitions of niche terminology, relationships between underlying resources—and make it explicit. Avoid ambiguity by clearly describing (and enforcing with strict data models) expected inputs and outputs. In particular, input parameters should be unambiguously named: instead of a parameter named `user`, try a parameter named `user_id`.

通过评估体系，你可以更有把握地衡量提示工程的效果。即使对工具描述的微小改进也能带来显著的性能提升。Claude Sonnet 3.5 在 SWE-bench Verified 评估上取得了最先进的性能，正是在我们对工具描述进行精确改进之后——大幅降低了错误率并提高了任务完成率。

With your evaluation you can measure the impact of your prompt engineering with greater confidence. Even small refinements to tool descriptions can yield dramatic improvements. Claude Sonnet 3.5 achieved state-of-the-art performance on the SWE-bench Verified evaluation after we made precise refinements to tool descriptions, dramatically reducing error rates and improving task completion.

你可以在我们的[开发者指南](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview)中找到工具定义的其他最佳实践。如果你是为 Claude 构建工具，我们还建议阅读有关工具如何动态加载到 Claude 系统提示中的内容。最后，如果你正在为 MCP 服务器编写工具，[工具注解](https://modelcontextprotocol.io/docs/concepts/tool-annotations)有助于披露哪些工具需要开放网络访问或会进行破坏性更改。

You can find other best practices for tool definitions in our [Developer Guide](https://docs.anthropic.com/en/docs/build-with-claude/tool-use/overview). If you're building tools for Claude, we also recommend reading about how tools are dynamically loaded into Claude's system prompt. Lastly, if you're writing tools for an MCP server, [tool annotations](https://modelcontextprotocol.io/docs/concepts/tool-annotations) help disclose which tools require open-world access or make destructive changes.

## 展望未来

Looking ahead

要为智能体构建高效工具，我们需要将软件开发实践从可预测的确定性模式转向非确定性模式。

To build effective tools for agents, we need to re-orient our software development practices from predictable, deterministic patterns to non-deterministic ones.

通过我们在本文中描述的迭代式、评估驱动的流程，我们识别出了使工具成功的一致模式：高效的工具具有明确而清晰的定义，审慎地使用智能体上下文，能在多样化的工作流中组合使用，并使智能体能直觉性地解决现实世界任务。

Through the iterative, evaluation-driven process we've described in this post, we've identified consistent patterns in what makes tools successful: Effective tools are intentionally and clearly defined, use agent context judiciously, can be combined together in diverse workflows, and enable agents to intuitively solve real-world tasks.

未来，我们预期智能体与世界交互的具体机制将不断演进——从 MCP 协议的更新到底层 LLM 自身的升级。通过系统性的、评估驱动的方法来改进智能体工具，我们可以确保随着智能体变得更强大，它们使用的工具也将随之进化。

In the future, we expect the specific mechanisms through which agents interact with the world to evolve—from updates to the MCP protocol to upgrades to the underlying LLMs themselves. With a systematic, evaluation-driven approach to improving tools for agents, we can ensure that as agents become more capable, the tools they use will evolve alongside them.

## 致谢

Acknowledgements

本文由 Ken Aizawa 撰写，得到了来自研究团队（Barry Zhang、Zachary Witten、Daniel Jiang、Sami Al-Sheikh、Matt Bell、Maggie Vo）、MCP 团队（Theodora Chu、John Welsh、David Soria Parra、Adam Jones）、产品工程团队（Santiago Seira）、市场团队（Molly Vorwerck）、设计团队（Drew Roper）和应用 AI 团队（Christian Ryan、Alexander Bricken）的宝贵贡献。

Written by Ken Aizawa with valuable contributions from colleagues across Research (Barry Zhang, Zachary Witten, Daniel Jiang, Sami Al-Sheikh, Matt Bell, Maggie Vo), MCP (Theodora Chu, John Welsh, David Soria Parra, Adam Jones), Product Engineering (Santiago Seira), Marketing (Molly Vorwerck), Design (Drew Roper), and Applied AI (Christian Ryan, Alexander Bricken).

---

<sup>1</sup> 不包括底层 LLM 自身的训练。

<sup>1</sup> Beyond training the underlying LLMs themselves.
