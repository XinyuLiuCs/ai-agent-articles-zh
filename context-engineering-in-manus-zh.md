# Manus 中的上下文工程

Context Engineering in Manus---
https://rlancemartin.github.io/2025/10/15/manus/

2025年10月15日

Oct 15, 2025

Lance Martin

## 为什么需要上下文工程

Why Context Engineering

本周早些时候，我与 Manus 联合创始人兼首席战略官 Yichao "Peak" Ji 进行了一场网络研讨会。你可以在这里观看视频，这里查看我的幻灯片，这里查看 Peak 的幻灯片。以下是我的笔记。

Earlier this week, I had a webinar with Manus co-founder and CSO Yichao "Peak" Ji. You can see the video here, my slides here, and Peak's slides here. Below are my notes.

Anthropic 将智能体定义为由大语言模型（LLM）指导自身流程和工具使用的系统，控制如何完成任务。简而言之，就是 LLM 在循环中调用工具。

Anthropic defines agents as systems where LLMs direct their own processes and tool usage, maintaining control over how they accomplish tasks. In short, it's an LLM calling tools in a loop.

Manus 是最受欢迎的通用型消费级智能体之一。典型的 Manus 任务会使用 50 次工具调用。如果没有上下文工程，这些工具调用结果会在 LLM 的上下文窗口中累积。随着上下文窗口的填充，许多人观察到 LLM 性能会下降。

Manus is one of the most popular general-purpose consumer agents. The typical Manus task uses 50 tool calls. Without context engineering, these tool call results would accumulate in the LLM context window. As the context window fills, many have observed that LLM performance degrades.

例如，Chroma 对上下文退化进行了出色的研究，Anthropic 也解释了上下文增长如何消耗 LLM 的注意力预算。因此，在构建智能体时，仔细管理进入 LLM 上下文窗口的内容至关重要。Karpathy 对此做出了清晰的阐述：

For example, Chroma has a great study on context rot and Anthropic has explained how growing context depletes an LLM's attention budget. So, it's important to carefully manage what goes into the LLM's context window when building agents. Karpathy laid this out clearly:

上下文工程是一门精妙的艺术与科学，旨在为（智能体轨迹中的）下一步填充恰到好处的信息到上下文窗口中。

Context engineering is the delicate art and science of filling the context window with just the right information for the next step (in an agent's trajectory)

## 上下文工程方法

Context Engineering Approaches

每个 Manus 会话使用专用的基于云的虚拟机，为智能体提供一台带有文件系统的虚拟计算机、导航文件系统的工具，以及在沙箱环境中执行命令（例如，提供的实用工具和标准 shell 命令）的能力。

Each Manus session uses a dedicated cloud-based virtual machine, giving the agent a virtual computer with a filesystem, tools to navigate it, and the ability to execute commands (e.g., provided utilities and standard shell commands) in that sandbox environment.

在这个沙箱中，Manus 使用三种主要的上下文工程策略，这些策略与 Anthropic 在此处介绍的方法一致，我也在许多项目中见过：

In this sandbox, Manus uses three primary strategies for context engineering, which align with approaches Anthropic covers here and I've seen in across many projects:

减少上下文

Reduce Context

卸载上下文

Offload Context

隔离上下文

Isolate Context

## 上下文减少

Context Reduction

Manus 中的工具调用具有"完整"和"紧凑"两种表示形式。完整版本包含工具调用的原始内容（例如，完整的搜索工具结果），存储在沙箱中（例如，文件系统）。紧凑版本存储对完整结果的引用（例如，文件路径）。

Tool calls in Manus have a "full" and "compact" representation. The full version contains the raw content from tool invocation (e.g., a complete search tool result), which is stored in the sandbox (e.g., filesystem). The compact version stores a reference to the full result (e.g., a file path).

Manus 对较旧的（"过时的"）工具结果应用压缩。这只是意味着将完整的工具结果替换为紧凑版本。这允许智能体在需要时仍然可以获取完整结果，但通过移除智能体已经用于决策的"过时"结果来节省 token。

Manus applies compaction to older ("stale") tool results. This just means swapping out the full tool result for the compact version. This allows the agent to still fetch the full result if ever needed, but saves tokens by removing "stale" results that the agent has already used to make decisions.

较新的工具结果保持完整形式，以指导智能体的下一个决策。这似乎是一种普遍有用的上下文减少策略，我注意到它与 Anthropic 的上下文编辑功能类似：

Newer tool results remain in full to guide the agent's next decision. This seems to be a generally useful strategy for context reduction, and I notice that it's similar to Anthropic's context editing feature:

上下文编辑会在接近 token 限制时自动清除上下文窗口中的过时工具调用和结果。随着智能体执行任务并累积工具结果，上下文编辑会移除过时内容，同时保持对话流程，有效地延长智能体在无需手动干预的情况下运行的时间。

Context editing automatically clears stale tool calls and results from within the context window when approaching token limits. As your agent executes tasks and accumulates tool results, context editing removes stale content while preserving the conversation flow, effectively extending how long agents can run without manual intervention.

当压缩达到收益递减的程度时（见下图），Manus 会对轨迹应用摘要。摘要使用完整的工具结果生成，Manus 使用模式（schema）来定义摘要字段。这为任何智能体轨迹创建了一致的摘要对象。

When compaction reaches diminishing returns (see figure below), Manus applies summarization to the trajectory. Summaries are generated using full tool results and Manus uses a schema to define the summary fields. This creates a consistent summary object for any agent trajectory.

## 上下文隔离

Context Isolation

Manus 对多智能体采取务实的方法，避免拟人化的分工。虽然人类由于认知限制而按角色（设计师、工程师、项目经理）组织，但 LLM 不一定有这些相同的约束。

Manus takes a pragmatic approach to multi-agent, avoiding anthropomorphized divisions of labor. While humans organize by role (designer, engineer, project manager) due to cognitive limitations, LLMs don't necessarily share these same constraints.

考虑到这一点，Manus 中子智能体的主要目标是隔离上下文。例如，如果有一个任务要完成，Manus 会将该任务分配给一个拥有自己上下文窗口的子智能体。

With this in mind, the primary goal of sub-agents in Manus is to isolate context. For example, if there's a task to be done, Manus will assign that task to a sub-agent with its own context window.

Manus 使用多智能体架构，包括一个分配任务的规划器（planner）、一个审查对话并确定应该保存在文件系统中的内容的知识管理器，以及一个执行规划器分配任务的执行器（executor）子智能体。

Manus uses multi-agent with a planner that assigns tasks, a knowledge manager that reviews conversations and determines what should be saved in the filesystem, and an executor sub-agent that performs tasks assigned by the planner.

**Manus 最初使用 todo.md 进行任务规划，但发现大约三分之一的操作都花在更新待办事项列表上，浪费了宝贵的 token。他们转而使用专门的规划器智能体，该智能体调用执行器子智能体来执行任务。**

Manus initially used a todo.md for task planning, but found that roughly one-third of all actions were spent updating the todo list, wasting valuable tokens. They shifted to a dedicated planner agent that calls executor sub-agents to perform tasks.

在最近的播客中，Erik Schluntz（Anthropic 多智能体研究）提到，他们也类似地设计多智能体系统，使用规划器分配任务，并使用函数调用作为通信协议来启动子智能体。Erik 以及 Walden Yan（Cognition）提出的一个核心挑战是规划器和子智能体之间的上下文共享。

In a recent podcast, Erik Schluntz (multi-agent research at Anthropic) mentioned that they similarly design multi-agent systems with a planner to assign tasks and use function calling as the communication protocol to initiate sub-agents. A central challenge raised by Erik as well as Walden Yan (Cognition) is context sharing between planner and sub-agents.

Manus 通过两种方式解决这个问题。对于简单任务（例如，规划器只需要子智能体的输出的离散任务），**规划器只需创建指令并通过函数调用将其传递给子智能体**。这类似于 Claude Code 的任务工具。

Manus addresses this in two ways. For simple tasks (e.g., a discrete task where the planner only needs the output of the sub-agent), the planner simply creates instructions and passes them to the sub-agent via the function call. This resembles Claude Code's task tool.

对于更复杂的任务（例如，子智能体需要写入规划器也使用的文件），规划器与子智能体共享其完整上下文。子智能体仍然有自己的操作空间（工具）和指令，但接收规划器也可以访问的完整上下文。

For more complex tasks (e.g., the sub-agent needs to write to files that the planner also uses), the planner shares its full context with the sub-agent. The sub-agent still has its own action space (tools) and instructions, but receives the full context that the planner also has access to.

在这两种情况下，规划器都定义子智能体的输出模式（schema）。子智能体有一个提交结果工具，用于在将结果返回给规划器之前填充此模式，Manus 使用受约束解码来确保输出符合定义的模式。

In both cases, the planner defines the sub-agent's output schema. Sub-agents have a submit results tool to populate this schema before returning results to the planner and Manus uses constrained decoding to ensure output adheres to the defined schema.

## 上下文卸载

Context Offloading

### 工具定义

Tools Definitions

我们经常希望智能体能够执行各种各样的操作。当然，我们可以向 LLM 绑定大量工具，并提供关于如何使用所有工具的详细说明。但是，工具描述会消耗宝贵的 token，而且许多（通常重叠或模糊的）工具可能会导致模型混淆。

We often want agents that can perform a wide range of actions. We can, of course, bind a large collection of tools to the LLM and provide detailed instructions on how to use all of them. But, tool descriptions use valuable tokens and many (often overlapping or ambiguous) tools can cause model confusion.

我看到的一个趋势是，**智能体使用一小组通用工具**，使智能体能够访问计算机。例如，仅使用 Bash 工具和几个访问文件系统的工具，智能体就可以执行各种操作！

A trend I'm seeing is that agents use a small set of general tools that give the agent access to a computer. For example, with only a Bash tool and a few tools to access a filesystem, an agent can perform a wide range of actions!

Manus 将其视为具有函数/工具调用和虚拟计算机沙箱的分层操作空间。Peak 提到 Manus 使用一小组（< 20 个）原子函数；这包括 **Bash 工具、管理文件系统的工具和代码执行工具**等。

Manus thinks about this as a layered action space with function/tool calling and its virtual computer sandbox. Peak mentioned that Manus uses a small set (< 20) of atomic functions; this includes things like a Bash tool, tools to manage the filesystem, and a code execution tool.

Manus 将大部分操作卸载到沙箱层，而不是使函数调用层膨胀。Manus 可以使用其 Bash 工具直接在沙箱中执行许多实用程序，并且 MCP 工具通过智能体也可以使用 Bash 工具执行的 CLI 公开。

Rather than bloating the function calling layer, Manus offloads most actions to the sandbox layer. Manus can execute many utilities directly in the sandbox with its Bash tool and MCP tools are exposed through a CLI that the agent can also execute using the Bash tool.

Claude 的技能（skills）功能使用类似的思路：技能存储在文件系统中，而不是作为绑定工具，Claude 只需要几个简单的函数调用（Bash、文件系统）即可逐步发现和使用它们。

Claude's skills feature uses a similar idea: skills are stored in the filesystem, not as bound tools, and Claude only needs a few simple function calls (Bash, file system) to progressively discover and use them.

渐进式披露是使 Agent Skills 灵活且可扩展的核心设计原则。就像一本组织良好的手册，从目录开始，然后是特定章节，最后是详细附录，技能让 Claude 只在需要时加载信息……拥有文件系统和代码执行工具的智能体在处理特定任务时不需要将整个技能读入其上下文窗口。

Progressive disclosure is the core design principle that makes Agent Skills flexible and scalable. Like a well-organized manual that starts with a table of contents, then specific chapters, and finally a detailed appendix, skills let Claude load information only as needed … agents with a filesystem and code execution tools don't need to read the entirety of a skill into their context window when working on a particular task.

### 工具结果

Tool Results

由于 Manus 可以访问文件系统，它也可以卸载上下文（例如，工具结果）。如上所述，这对于上下文减少至关重要；工具结果被卸载到文件系统以生成紧凑版本，这用于从智能体的上下文窗口中修剪过时的 token。与 Claude Code 类似，Manus 使用基本实用程序（例如 glob 和 grep）搜索文件系统，而不需要索引（例如，向量存储）。

Because Manus has access to a filesystem, it can also offload context (e.g., tool results). As explained above, this is central for context reduction; tool results are offloaded to the filesystem in order to produce the compact version and this is used to prune stale tokens from the agent's context window. Similar to Claude Code, Manus uses basic utilities (e.g., glob and grep) to search the filesystem without the need for indexing (e.g., vectorstores).

## 模型选择

Model Choice

Manus 不会固定使用单一模型，而是使用任务级路由：它可能使用 Claude 进行编码，使用 Gemini 处理多模态任务，或使用 OpenAI 进行数学和推理。总体而言，Manus 的模型选择方法由成本考虑驱动，其中 KV 缓存效率起着核心作用。

Rather than committing to a single model, Manus uses task-level routing: it might use Claude for coding, Gemini for multi-modal tasks, or OpenAI for math and reasoning. Broadly, Manus's approach to model selection is driven by cost considerations, with KV cache efficiency playing a central role.

Manus 使用缓存（例如，用于系统指令、较旧的工具结果等）来减少多个智能体回合的成本和延迟。Peak 提到，分布式 KV 缓存基础设施对于开源模型来说实施起来具有挑战性，但前沿供应商支持良好。这种缓存支持可以使前沿模型在实践中对于某些（智能体）用例更便宜。

Manus uses caching (e.g., for system instructions, older tool results, etc) to reduce both cost and latency across many agent turns. Peak mentioned that distributed KV cache infrastructure is challenging to implement with open source models, but is well-supported by frontier providers. This caching support can make frontier models cheaper for certain (agent) use-cases in practice.

## 牢记"痛苦的教训"进行构建

Build with the Bitter Lesson in Mind

我们结束讨论时谈到了"痛苦的教训"（Bitter Lesson）。我一直对它对 AI 工程的影响很感兴趣。Boris Cherny（Claude Code 的创建者）提到，痛苦的教训影响了他保持 Claude Code 不持有观点的决定，使其更容易适应模型改进。

We closed the discussion talking about the Bitter Lesson. I've been interested in its implications for AI engineering. Boris Cherny (creator of Claude Code) mentioned that The Bitter Lesson influenced his decision to keep Claude Code unopinionated, making it easier to adapt to model improvements.

基于不断改进的模型进行构建意味着接受持续的变化。Peak 提到，自 3 月推出以来，Manus 已经重构了五次！

Building on constantly improving models means accepting constant change. Peak mentioned that Manus has been refactored five times since their launch in March!

此外，Peak 警告说，随着模型的进步，智能体的框架可能会限制性能；这正是痛苦的教训所指出的挑战。我们添加结构是为了在某个时间点提高性能，但随着计算能力（模型）的增长，这种结构可能会限制性能。

In addition, Peak warned that the agent's harness can limit performance as models advance; this is exactly the challenge called out by the Bitter Lesson. We add structure to improve performance at a point in time, but this structure can limit performance as compute (models) grows.

为了防止这种情况，Peak 建议在不同的模型强度下运行智能体评估。**如果性能没有随着更强的模型而提高，你的框架可能正在阻碍智能体。**这可以帮助测试你的框架是否"经得起未来考验"。

To guard against this, Peak suggested running agent evaluations across varying model strengths. If performance doesn't improve with stronger models, your harness may be hobbling the agent. This can help test whether your harness is "future proof".

Hyung Won Chung（OpenAI/MSL）关于此主题的演讲进一步强调了随着模型改进，需要持续重新评估结构（例如，你的框架/假设）的必要性。

Hyung Won Chung's (OpenAI/MSL) talk on this topic further emphasizes the need to consistently re-evaluate structure (e.g., your harness / assumptions) as models improve.

为给定水平的计算和可用数据添加所需的结构。稍后将其移除，因为这些捷径将成为进一步改进的瓶颈。

Add structures needed for the given level of compute and data available. Remove them later, because these shortcuts will bottleneck further improvement.

## 结论

Conclusions

让智能体访问计算机（例如，文件系统、终端、实用程序）是我们在许多智能体中看到的常见模式，包括 Manus。它实现了几种上下文工程策略：

Giving agents access to a computer (e.g., filesystem, terminal, utilities) is a common pattern we see across many agents, including Manus. It enables a few context engineering strategies:

### 1. 卸载上下文

1. Offload Context

外部存储工具结果：将完整的工具结果保存到文件系统（而不是在上下文中），并根据需要使用 glob 和 grep 等实用程序访问

Store tool results externally: Save full tool results to the filesystem (not in context) and access on demand with utilities like glob and grep

将操作推送到沙箱：使用一小组函数调用（Bash、文件系统访问），可以在沙箱中执行许多实用程序，而不是将每个实用程序都绑定为工具

Push actions to the sandbox: Use a small set of function calls (Bash, filesystem access) that can execute many utilities in the sandbox rather than binding every utility as a tool

### 2. 减少上下文

2. Reduce Context

压缩过时结果：随着上下文填充，用引用（例如，文件路径）替换较旧的工具结果；保持最近的结果完整以指导下一个决策

Compact stale results: Replace older tool results with references (e.g., file paths) as context fills; keep recent results in full to guide the next decision

在需要时进行摘要：一旦压缩达到收益递减，对完整轨迹应用基于模式的摘要

Summarize when needed: Once compaction reaches diminishing returns, apply schema-based summarization to the full trajectory

### 3. 隔离上下文

3. Isolate Context

对离散任务使用子智能体：将任务分配给拥有自己上下文窗口的子智能体，主要是为了隔离上下文（而不是按角色分工）

Use sub-agents for discrete tasks: Assign tasks to sub-agents with their own context windows, primarily to isolate context (not to divide labor by role)

有意识地共享上下文：对于简单任务只传递指令；对于子智能体需要更多上下文的复杂任务传递完整上下文（例如，轨迹和共享文件系统）

Share context deliberately: Pass only instructions for simple tasks; pass full context (e.g., trajectory and shared filesystem) for complex tasks where sub-agents need more context

最后一个考虑因素是确保你的框架不会随着模型改进而限制性能（即，要"吸取痛苦的教训"）。跨模型强度进行测试以验证这一点。简单、不持有观点的设计通常能更好地适应模型改进。最后，不要害怕随着模型的改进重新构建你的智能体（Manus 自 3 月以来重构了 5 次）！

A final consideration is to ensure your harness is not limiting performance as models improve (e.g., be "Bitter Lesson-pilled"). Test across model strengths to verify this. Simple, unopinionated designs often adapt better to model improvements. Finally, don't be afraid to re-build your agent as models improve (Manus refactored 5 times since March)!
