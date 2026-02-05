# 智能体的上下文工程

Context Engineering for Agents---
https://rlancemartin.github.io/2025/06/23/context_engineering/

2025年6月23日

Jun 23, 2025

Lance Martin

## TL;DR

TL;DR

智能体需要上下文来执行任务。上下文工程是一门艺术与科学，旨在为智能体轨迹的每一步填充恰到好处的信息到上下文窗口中。在这篇文章中，我将上下文工程分为几种常见策略，这些策略在当今许多流行的智能体中都能看到。

Agents need context to perform tasks. Context engineering is the art and science of filling the context window with just the right information at each step of an agent's trajectory. In this post, I group context engineering into a few common strategies seen across many popular agents today.

## 上下文工程

Context Engineering

正如 Andrej Karpathy 所说，LLM 就像一种新型操作系统。LLM 就像 CPU，其上下文窗口就像 RAM，充当模型的工作内存。就像 RAM 一样，LLM 上下文窗口处理各种上下文源的容量有限。正如操作系统管理 CPU 的 RAM 中放入什么内容一样，"上下文工程"扮演着类似的角色。Karpathy 对此总结得很好：

As Andrej Karpathy puts it, LLMs are like a new kind of operating system. The LLM is like the CPU and its context window is like the RAM, serving as the model's working memory. Just like RAM, the LLM context window has limited capacity to handle various sources of context. And just as an operating system curates what fits into a CPU's RAM, "context engineering" plays a similar role. Karpathy summarizes this well:

[上下文工程是] "……为下一步向上下文窗口填充恰到好处信息的精妙艺术与科学。"

[Context engineering is the] "…delicate art and science of filling the context window with just the right information for the next step."

在构建 LLM 应用时，我们需要管理哪些类型的上下文？上下文工程是一个涵盖几种不同上下文类型的总括术语：

What are the types of context that we need to manage when building LLM applications? Context engineering is an umbrella that applies across a few different context types:

**指令** – 提示、记忆、少样本示例、工具描述等

Instructions – prompts, memories, few‑shot examples, tool descriptions, etc

**知识** – 事实、记忆等

Knowledge – facts, memories, etc

**工具** – 工具调用的反馈

Tools – feedback from tool calls

## 智能体的上下文工程

Context Engineering for Agents

今年，随着 LLM 在推理和工具调用方面变得更好，人们对智能体的兴趣大幅增长。智能体交错进行 LLM 调用和工具调用，通常用于长期运行的任务。

This year, interest in agents has grown tremendously as LLMs get better at reasoning and tool calling. Agents interleave LLM invocations and tool calls, often for long-running tasks.

然而，长期运行的任务和来自工具调用的累积反馈意味着智能体通常会使用大量 token。这可能导致许多问题：可能超出上下文窗口的大小，使成本/延迟激增，或降低智能体性能。Drew Breunig 很好地概述了较长上下文可能导致性能问题的几种具体方式，包括：

However, long-running tasks and accumulating feedback from tool calls mean that agents often utilize a large number of tokens. This can cause numerous problems: it can exceed the size of the context window, balloon cost / latency, or degrade agent performance. Drew Breunig nicely outlined a number of specific ways that longer context can cause perform problems, including:

**上下文中毒**：当幻觉进入上下文时

Context Poisoning: When a hallucination makes it into the context

**上下文分心**：当上下文压倒训练时

Context Distraction: When the context overwhelms the training

**上下文混淆**：当多余的上下文影响响应时

Context Confusion: When superfluous context influences the response

**上下文冲突**：当上下文的部分内容不一致时

Context Clash: When parts of the context disagree

考虑到这一点，Cognition 指出了上下文工程的重要性：

With this in mind, Cognition called out the importance of context engineering:

"上下文工程"……实际上是构建 AI 智能体的工程师的头号工作。

"Context engineering" … is effectively the #1 job of engineers building AI agents.

Anthropic 也清楚地表明了这一点：

Anthropic also laid it out clearly:

智能体通常进行跨越数百轮的对话，需要仔细的上下文管理策略。

Agents often engage in conversations spanning hundreds of turns, requiring careful context management strategies.

那么，今天人们如何应对这一挑战？我将方法分为 4 个类别——写入、选择、压缩和隔离——并在下面给出每个类别的一些示例。

So, how are people tackling this challenge today? I group approaches into 4 buckets — write, select, compress, and isolate — and give some examples of each one below.

## 写入上下文

Write Context

写入上下文意味着将其保存在上下文窗口之外，以帮助智能体执行任务。

Writing context means saving it outside the context window to help an agent perform a task.

### 草稿本

Scratchpads

当人类解决任务时，我们会做笔记并记住事情以备将来相关任务使用。智能体也在获得这些能力！通过"草稿本"做笔记是在智能体执行任务时持久化信息的一种方法。核心思想是将信息保存在上下文窗口之外，以便智能体可以使用。Anthropic 的多智能体研究员展示了一个清晰的例子：

When humans solve tasks, we take notes and remember things for future, related tasks. Agents are also gaining these capabilities! Note-taking via a "scratchpad" is one approach to persist information while an agent is performing a task. The central idea is to save information outside of the context window so that it's available to the agent. Anthropic's multi-agent researcher illustrates a clear example of this:

LeadResearcher 首先思考方法并将其计划保存到内存中以持久化上下文，因为如果上下文窗口超过 200,000 个 token，它将被截断，保留计划很重要。

The LeadResearcher begins by thinking through the approach and saving its plan to Memory to persist the context, since if the context window exceeds 200,000 tokens it will be truncated and it is important to retain the plan.

草稿本可以通过几种不同的方式实现。它们可以是简单地写入文件的工具调用。它也可以只是运行时状态对象中的一个字段，在会话期间持久存在。无论哪种情况，草稿本都让智能体保存有用的信息以帮助它们完成任务。

Scratchpads can be implemented in a few different ways. They can be a tool call that simply writes to a file. It could also just be a field in a runtime state object that persists during the session. In either case, scratchpads let agents save useful information to help them accomplish a task.

### 记忆

Memories

草稿本帮助智能体在给定会话中解决任务，但有时智能体受益于在许多会话中记住事情。Reflexion 引入了在每个智能体回合后进行反思并重用这些自生成记忆的想法。Generative Agents 创建了从过去智能体反馈集合中定期综合的记忆。

Scratchpads help agents solve a task within a given session, but sometimes agents benefit from remembering things across many sessions. Reflexion introduced the idea of reflection following each agent turn and re-using these self-generated memories. Generative Agents created memories synthesized periodically from collections of past agent feedback.

这些概念进入了像 ChatGPT、Cursor 和 Windsurf 这样的流行产品，它们都有基于用户-智能体交互自动生成长期记忆的机制。

These concepts made their way into popular products like ChatGPT, Cursor, and Windsurf, which all have mechanisms to auto-generate long-term memories based on user-agent interactions.

## 选择上下文

Select Context

选择上下文意味着将其拉入上下文窗口以帮助智能体执行任务。

Selecting context means pulling it into the context window to help an agent perform a task.

### 草稿本

Scratchpad

从草稿本中选择上下文的机制取决于草稿本的实现方式。如果它是一个工具，那么智能体可以通过进行工具调用来简单地读取它。如果它是智能体运行时状态的一部分，那么开发者可以选择在每一步向智能体公开状态的哪些部分。这提供了对在后续轮次向 LLM 公开草稿本上下文的细粒度控制。

The mechanism for selecting context from a scratchpad depends upon how the scratchpad is implemented. If it's a tool, then an agent can simply read it by making a tool call. If it's part of the agent's runtime state, then the developer can choose what parts of state to expose to an agent each step. This provides a fine-grained level of control for exposing scratchpad context to the LLM at later turns.

### 记忆

Memories

如果智能体有能力保存记忆，它们也需要有选择与正在执行的任务相关的记忆的能力。这可能有几个原因很有用。智能体可能选择少样本示例（情景记忆）作为期望行为的示例，选择指令（程序性记忆）来引导行为，或选择事实（语义记忆）来为智能体提供任务相关的上下文。

If agents have the ability to save memories, they also need the ability to select memories relevant to the task they are performing. This can be useful for a few reasons. Agents might select few-shot examples (episodic memories) for examples of desired behavior, instructions (procedural memories) to steer behavior, or facts (semantic memories) give the agent task-relevant context.

一个挑战是确保选择相关的记忆。一些流行的智能体只是使用一组狭窄的文件，这些文件总是被拉入上下文。例如，许多代码智能体使用文件来保存指令（"程序性"记忆）或在某些情况下保存示例（"情景"记忆）。Claude Code 使用 CLAUDE.md。Cursor 和 Windsurf 使用规则文件。

One challenge is ensuring that relevant memories are selected. Some popular agents simply use a narrow set of files that are always pulled into context. For example, many code agent use files to save instructions ("procedural" memories) or, in some cases, examples ("episodic" memories). Claude Code uses CLAUDE.md. Cursor and Windsurf use rules files.

但是，如果智能体存储大量事实和/或关系（例如，语义记忆），选择就更困难了。ChatGPT 是一个流行产品的好例子，它存储并从大量特定于用户的记忆中进行选择。

But, if an agent is storing a larger collection of facts and / or relationships (e.g., semantic memories), selection is harder. ChatGPT is a good example of a popular product that stores and selects from a large collection of user-specific memories.

嵌入和/或知识图谱用于记忆索引通常用于辅助选择。尽管如此，记忆选择仍然具有挑战性。在 AIEngineer World's Fair 上，Simon Willison 分享了一个记忆选择出错的例子：ChatGPT 从记忆中获取了他的位置并意外地将其注入到请求的图像中。这种意外或不需要的记忆检索可能会让一些用户感觉上下文窗口"不再属于他们"！

Embeddings and / or knowledge graphs for memory indexing are commonly used to assist with selection. Still, memory selection is challenging. At the AIEngineer World's Fair, Simon Willison shared an example of memory selection gone wrong: ChatGPT fetched his location from memories and unexpectedly injected it into a requested image. This type of unexpected or undesired memory retrieval can make some users feel like the context window "no longer belongs to them"!

### 工具

Tools

智能体使用工具，但如果提供太多工具，它们可能会变得过载。这通常是因为工具描述可能重叠，导致模型对使用哪个工具感到困惑。一种方法是对工具描述应用 RAG（检索增强生成），以便根据语义相似性为任务获取最相关的工具。一些最近的论文表明，这将工具选择准确性提高了 3 倍。

Agents use tools, but can become overloaded if they are provided with too many. This is often because the tool descriptions can overlap, causing model confusion about which tool to use. One approach is to apply RAG (retrieval augmented generation) to tool descriptions in order to fetch the most relevant tools for a task based upon semantic similarity. Some recent papers have shown that this improves tool selection accuracy by 3-fold.

### 知识

Knowledge

RAG 是一个丰富的话题，可能是一个核心的上下文工程挑战。代码智能体是大规模生产中 RAG 的一些最佳示例。Windsurf 的 Varun 很好地捕捉了其中一些挑战：

RAG is a rich topic and can be a central context engineering challenge. Code agents are some of the best examples of RAG in large-scale production. Varun from Windsurf captures some of these challenges well:

索引代码 ≠ 上下文检索……[我们正在进行索引和嵌入搜索……[使用] AST 解析代码并沿着语义有意义的边界进行分块……随着代码库规模的增长，嵌入搜索作为检索启发式变得不可靠……我们必须依赖 grep/文件搜索、基于知识图谱的检索等技术的组合，以及……重新排序步骤，其中[上下文]按相关性顺序排名。

Indexing code ≠ context retrieval … [We are doing indexing & embedding search … [with] AST parsing code and chunking along semantically meaningful boundaries … embedding search becomes unreliable as a retrieval heuristic as the size of the codebase grows … we must rely on a combination of techniques like grep/file search, knowledge graph based retrieval, and … a re-ranking step where [context] is ranked in order of relevance.

## 压缩上下文

Compressing Context

压缩上下文涉及仅保留执行任务所需的 token。

Compressing context involves retaining only the tokens required to perform a task.

### 上下文摘要

Context Summarization

智能体交互可以跨越数百轮并使用消耗大量 token 的工具调用。摘要是管理这些挑战的一种常见方式。如果你使用过 Claude Code，你就会看到这一点。Claude Code 在你超过上下文窗口的 95% 后运行"自动压缩"，它将摘要用户-智能体交互的完整轨迹。这种跨智能体轨迹的压缩可以使用各种策略，如递归或分层摘要。

Agent interactions can span hundreds of turns and use token-heavy tool calls. Summarization is one common way to manage these challenges. If you've used Claude Code, you've seen this in action. Claude Code runs "auto-compact" after you exceed 95% of the context window and it will summarize the full trajectory of user-agent interactions. This type of compression across an agent trajectory can use various strategies such as recursive or hierarchical summarization.

在智能体设计的某些点添加摘要也可能很有用。例如，它可以用于后处理某些工具调用（例如，消耗大量 token 的搜索工具）。作为第二个例子，Cognition 提到在智能体-智能体边界进行摘要，以减少知识交接期间的 token。如果需要捕获特定事件或决策，摘要可能是一个挑战。Cognition 为此使用了微调模型，这强调了这一步可以投入多少工作。

It can also be useful to add summarization at points in an agent's design. For example, it can be used to post-process certain tool calls (e.g., token-heavy search tools). As a second example, Cognition mentioned summarization at agent-agent boundaries to reduce tokens during knowledge hand-off. Summarization can be a challenge if specific events or decisions need to be captured. Cognition uses a fine-tuned model for this, which underscores how much work can go into this step.

### 上下文修剪

Context Trimming

摘要通常使用 LLM 来提取最相关的上下文片段，而修剪通常可以过滤或如 Drew Breunig 所指出的"修剪"上下文。这可以使用硬编码的启发式方法，如从消息列表中删除较旧的消息。Drew 还提到了 Provence，一个用于问答的训练有素的上下文修剪器。

Whereas summarization typically uses an LLM to distill the most relevant pieces of context, trimming can often filter or, as Drew Breunig points out, "prune" context. This can use hard-coded heuristics like removing older messages from a message list. Drew also mentions Provence, a trained context pruner for Question-Answering.

## 隔离上下文

Isolating Context

隔离上下文涉及将其拆分以帮助智能体执行任务。

Isolating context involves splitting it up to help an agent perform a task.

### 多智能体

Multi-agent

隔离上下文最流行的方法之一是将其拆分到子智能体中。OpenAI Swarm 库的一个动机是"关注点分离"，其中智能体团队可以处理子任务。每个智能体都有一组特定的工具、指令和自己的上下文窗口。

One of the most popular ways to isolate context is to split it across sub-agents. A motivation for the OpenAI Swarm library was "separation of concerns", where a team of agents can handle sub-tasks. Each agent has a specific set of tools, instructions, and its own context window.

Anthropic 的多智能体研究员为此提出了理由：许多具有隔离上下文的智能体优于单智能体，主要是因为每个子智能体上下文窗口可以分配给更狭窄的子任务。正如博客所说：

Anthropic's multi-agent researcher makes a case for this: many agents with isolated contexts outperformed single-agent, largely because each subagent context window can be allocated to a more narrow sub-task. As the blog said:

[子智能体]并行操作，拥有各自的上下文窗口，同时探索问题的不同方面。

[Subagents operate] in parallel with their own context windows, exploring different aspects of the question simultaneously.

当然，多智能体的挑战包括 token 使用（例如，根据 Anthropic 报告，比聊天多 15 倍的 token）、需要仔细的提示工程来规划子智能体工作，以及子智能体的协调。

Of course, the challenges with multi-agent include token use (e.g., up to 15× more tokens than chat as reported by Anthropic), the need for careful prompt engineering to plan sub-agent work, and coordination of sub-agents.

### 使用环境隔离上下文

Context Isolation with Environments

HuggingFace 的深度研究员展示了另一个有趣的上下文隔离例子。大多数智能体使用工具调用 API，它返回 JSON 对象（工具参数），可以传递给工具（例如，搜索 API）以获取工具反馈（例如，搜索结果）。HuggingFace 使用 CodeAgent，它输出包含所需工具调用的代码。然后代码在沙箱中运行。然后将工具调用的选定上下文（例如，返回值）传递回 LLM。

HuggingFace's deep researcher shows another interesting example of context isolation. Most agents use tool calling APIs, which return JSON objects (tool arguments) that can be passed to tools (e.g., a search API) to get tool feedback (e.g., search results). HuggingFace uses a CodeAgent, which outputs code that contains the desired tool calls. The code then runs in a sandbox. Selected context (e.g., return values) from the tool calls is then passed back to the LLM.

这允许上下文在环境中与 LLM 隔离。Hugging Face 指出，这是隔离消耗大量 token 的对象的好方法：

This allows context to be isolated from the LLM in the environment. Hugging Face noted that this is a great way to isolate token-heavy objects in particular:

[代码智能体允许]更好地处理状态……需要存储这个图像/音频/其他以供以后使用？没问题，只需将其分配为你状态中的变量，你[稍后使用它]。

[Code Agents allow for] a better handling of state … Need to store this image / audio / other for later use? No problem, just assign it as a variable in your state and you [use it later].

### 状态

State

值得指出的是，智能体的运行时状态对象也可以是隔离上下文的好方法。这可以达到与沙箱相同的目的。状态对象可以使用模式（例如，Pydantic 模型）设计，该模式具有可以写入上下文的字段。模式的一个字段（例如，消息）可以在智能体的每一轮向 LLM 公开，但模式可以将其他字段中的信息隔离以供更有选择性地使用。

It's worth calling out that an agent's runtime state object can also be a great way to isolate context. This can serve the same purpose as sandboxing. A state object can be designed with a schema (e.g., a Pydantic model) that has fields that context can be written to. One field of the schema (e.g., messages) can be exposed to the LLM at each turn of the agent, but the schema can isolate information in other fields for more selective use.

## 结论

Conclusion

智能体上下文工程的模式仍在演变，但我们可以将常见方法分为 4 个类别——写入、选择、压缩和隔离：

Patterns for agent context engineering are still evolving, but we can group common approaches into 4 buckets — write, select, compress, and isolate —:

**写入上下文**意味着将其保存在上下文窗口之外，以帮助智能体执行任务。

Writing context means saving it outside the context window to help an agent perform a task.

**选择上下文**意味着将其拉入上下文窗口以帮助智能体执行任务。

Selecting context means pulling it into the context window to help an agent perform a task.

**压缩上下文**涉及仅保留执行任务所需的 token。

Compressing context involves retaining only the tokens required to perform a task.

**隔离上下文**涉及将其拆分以帮助智能体执行任务。

Isolating context involves splitting it up to help an agent perform a task.

理解和利用这些模式是当今构建有效智能体的核心部分。

Understanding and utilizing these patterns is a central part of building effective agents today.
