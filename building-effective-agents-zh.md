# 构建有效的智能体

*Building effective agents*

发布日期：2024年12月19日

Published Dec 19, 2024

我们与数十个跨行业构建 LLM 智能体的团队合作过。一致的是，最成功的实现并非使用复杂的框架或专门的库，而是采用简单、可组合的模式。

We've worked with dozens of teams building LLM agents across industries. Consistently, the most successful implementations use simple, composable patterns rather than complex frameworks.

在过去一年中，我们与数十个跨行业构建大语言模型（LLM）智能体的团队合作。一致的是，最成功的实现并非使用复杂的框架或专门的库，而是采用简单、可组合的模式。

Over the past year, we've worked with dozens of teams building large language model (LLM) agents across industries. Consistently, the most successful implementations weren't using complex frameworks or specialized libraries. Instead, they were building with simple, composable patterns.

在这篇文章中，我们分享了从与客户合作和自己构建智能体中学到的经验，并为开发者提供构建有效智能体的实用建议。

In this post, we share what we've learned from working with our customers and building agents ourselves, and give practical advice for developers on building effective agents.

## 什么是智能体？

What are agents?

"智能体"可以有多种定义。一些客户将智能体定义为完全自主的系统，能够在较长时间内独立运行，使用各种工具完成复杂任务。其他人则使用该术语来描述遵循预定义工作流的更具规定性的实现。在 Anthropic，我们将所有这些变体归类为智能体系统，但在工作流和智能体之间划分了重要的架构区别：

"Agent" can be defined in several ways. Some customers define agents as fully autonomous systems that operate independently over extended periods, using various tools to accomplish complex tasks. Others use the term to describe more prescriptive implementations that follow predefined workflows. At Anthropic, we categorize all these variations as agentic systems, but draw an important architectural distinction between workflows and agents:

**工作流**是通过预定义代码路径编排 LLM 和工具的系统。

**Workflows are systems where LLMs and tools are orchestrated through predefined code paths.**

另一方面，**智能体**是 LLM 动态指导自己的进程和工具使用的系统，保持对如何完成任务的控制。

Agents, on the other hand, are systems where LLMs dynamically direct their own processes and tool usage, maintaining control over how they accomplish tasks.

下面，我们将详细探讨这两种类型的智能体系统。在附录 1（"实践中的智能体"）中，我们描述了客户发现使用这些系统特别有价值的两个领域。

Below, we will explore both types of agentic systems in detail. In Appendix 1 ("Agents in Practice"), we describe two domains where customers have found particular value in using these kinds of systems.

## 何时使用（以及何时不使用）智能体

When (and when not) to use agents

在使用 LLM 构建应用程序时，我们建议找到尽可能简单的解决方案，只在需要时增加复杂性。这可能意味着根本不构建智能体系统。智能体系统通常以延迟和成本换取更好的任务性能，你应该考虑这种权衡何时有意义。

When building applications with LLMs, we recommend finding the simplest solution possible, and only increasing complexity when needed. This might mean not building agentic systems at all. Agentic systems often trade latency and cost for better task performance, and you should consider when this tradeoff makes sense.

当需要更多复杂性时，工作流为明确定义的任务提供可预测性和一致性，而当需要灵活性和模型驱动的大规模决策时，智能体是更好的选择。然而，对于许多应用程序，通过检索和上下文示例优化单个 LLM 调用通常就足够了。

When more complexity is warranted, workflows offer predictability and consistency for well-defined tasks, whereas agents are the better option when flexibility and model-driven decision-making are needed at scale. For many applications, however, optimizing single LLM calls with retrieval and in-context examples is usually enough.

## 何时以及如何使用框架

When and how to use frameworks

有许多框架使智能体系统更容易实现，包括：

There are many frameworks that make agentic systems easier to implement, including:

- Claude Agent SDK；
- AWS 的 Strands Agents SDK；
- Rivet，一个拖放式图形界面 LLM 工作流构建器；以及
- Vellum，另一个用于构建和测试复杂工作流的图形界面工具。

- The Claude Agent SDK;
- Strands Agents SDK by AWS;
- Rivet, a drag and drop GUI LLM workflow builder; and
- Vellum, another GUI tool for building and testing complex workflows.

这些框架通过简化标准的低级任务（如调用 LLM、定义和解析工具以及链接调用）使入门变得容易。然而，它们通常会创建额外的抽象层，可能会掩盖底层的提示和响应，使其更难调试。它们还可能会诱使你在更简单的设置就足够时增加复杂性。

These frameworks make it easy to get started by simplifying standard low-level tasks like calling LLMs, defining and parsing tools, and chaining calls together. However, they often create extra layers of abstraction that can obscure the underlying prompts ​​and responses, making them harder to debug. They can also make it tempting to add complexity when a simpler setup would suffice.

我们建议开发者从直接使用 LLM API 开始：许多模式可以用几行代码实现。如果你确实使用框架，请确保理解底层代码。对底层机制的错误假设是客户错误的常见来源。

We suggest that developers start by using LLM APIs directly: many patterns can be implemented in a few lines of code. If you do use a framework, ensure you understand the underlying code. Incorrect assumptions about what's under the hood are a common source of customer error.

请参阅我们的 cookbook 以获取一些示例实现。

See our cookbook for some sample implementations.

## 构建块、工作流和智能体

Building blocks, workflows, and agents

在本节中，我们将探讨我们在生产中看到的智能体系统的常见模式。我们将从基础构建块——增强型 LLM 开始，并逐步增加复杂性，从简单的组合工作流到自主智能体。

In this section, we'll explore the common patterns for agentic systems we've seen in production. We'll start with our foundational building block—the augmented LLM—and progressively increase complexity, from simple compositional workflows to autonomous agents.

### 构建块：增强型 LLM

Building block: The augmented LLM

智能体系统的基本构建块是一个增强了检索、工具和内存等功能的 LLM。我们目前的模型可以主动使用这些能力——生成自己的搜索查询、选择适当的工具并确定要保留的信息。

The basic building block of agentic systems is an LLM enhanced with augmentations such as retrieval, tools, and memory. Our current models can actively use these capabilities—generating their own search queries, selecting appropriate tools, and determining what information to retain.

**增强型 LLM**

The augmented LLM

我们建议关注实现的两个关键方面：根据你的特定用例定制这些能力，并确保它们为你的 LLM 提供易于使用且文档完善的接口。虽然有许多方法可以实现这些增强，但一种方法是通过我们最近发布的模型上下文协议（Model Context Protocol），它允许开发者通过简单的客户端实现与不断增长的第三方工具生态系统集成。

We recommend focusing on two key aspects of the implementation: tailoring these capabilities to your specific use case and ensuring they provide an easy, well-documented interface for your LLM. While there are many ways to implement these augmentations, one approach is through our recently released Model Context Protocol, which allows developers to integrate with a growing ecosystem of third-party tools with a simple client implementation.

在本文的其余部分，我们假设每次 LLM 调用都可以访问这些增强能力。

For the remainder of this post, we'll assume each LLM call has access to these augmented capabilities.

### 工作流：提示链

Workflow: Prompt chaining

提示链将任务分解为一系列步骤，其中每次 LLM 调用处理前一次的输出。你可以在任何中间步骤上添加程序化检查（见下图中的"门"）以确保流程仍在正轨上。

Prompt chaining decomposes a task into a sequence of steps, where each LLM call processes the output of the previous one. You can add programmatic checks (see "gate" in the diagram below) on any intermediate steps to ensure that the process is still on track.

**提示链工作流**

The prompt chaining workflow

**何时使用此工作流：** 当任务可以轻松、清晰地分解为固定子任务时，此工作流是理想选择。主要目标是通过使每次 LLM 调用成为更容易的任务来以延迟换取更高的准确性。

When to use this workflow: This workflow is ideal for situations where the task can be easily and cleanly decomposed into fixed subtasks. The main goal is to trade off latency for higher accuracy, by making each LLM call an easier task.

提示链有用的示例：

Examples where prompt chaining is useful:

- 生成营销文案，然后将其翻译成不同的语言。
- 编写文档大纲，检查大纲是否符合某些标准，然后根据大纲编写文档。

- Generating Marketing copy, then translating it into a different language.
- Writing an outline of a document, checking that the outline meets certain criteria, then writing the document based on the outline.

### 工作流：路由

Workflow: Routing

路由对输入进行分类，并将其定向到专门的后续任务。此工作流允许关注点分离和构建更专门的提示。如果没有此工作流，针对一种输入的优化可能会损害其他输入的性能。

Routing classifies an input and directs it to a specialized followup task. This workflow allows for separation of concerns, and building more specialized prompts. Without this workflow, optimizing for one kind of input can hurt performance on other inputs.

**路由工作流**

The routing workflow

**何时使用此工作流：** 路由适用于存在不同类别且更适合分别处理的复杂任务，以及可以准确处理分类的情况（无论是通过 LLM 还是更传统的分类模型/算法）。

When to use this workflow: Routing works well for complex tasks where there are distinct categories that are better handled separately, and where classification can be handled accurately, either by an LLM or a more traditional classification model/algorithm.

路由有用的示例：

Examples where routing is useful:

- 将不同类型的客户服务查询（一般问题、退款请求、技术支持）定向到不同的下游流程、提示和工具。
- 将简单/常见问题路由到更小、成本效益更高的模型（如 Claude Haiku 4.5），将困难/不寻常的问题路由到更强大的模型（如 Claude Sonnet 4.5）以优化最佳性能。

- Directing different types of customer service queries (general questions, refund requests, technical support) into different downstream processes, prompts, and tools.
- Routing easy/common questions to smaller, cost-efficient models like Claude Haiku 4.5 and hard/unusual questions to more capable models like Claude Sonnet 4.5 to optimize for best performance.

### 工作流：并行化

Workflow: Parallelization

LLM 有时可以同时处理任务并以程序化方式聚合其输出。此工作流，即并行化，体现在两个关键变体中：

LLMs can sometimes work simultaneously on a task and have their outputs aggregated programmatically. This workflow, parallelization, manifests in two key variations:

- **分段**：将任务分解为并行运行的独立子任务。
- **投票**：多次运行相同任务以获得多样化的输出。

- Sectioning: Breaking a task into independent subtasks run in parallel.
- Voting: Running the same task multiple times to get diverse outputs.

**并行化工作流**

The parallelization workflow

**何时使用此工作流：** 当划分的子任务可以并行化以提高速度时，或者当需要多个视角或尝试以获得更高置信度结果时，并行化是有效的。对于具有多个考虑因素的复杂任务，当每个考虑因素由单独的 LLM 调用处理时，LLM 通常表现更好，从而允许对每个特定方面集中关注。

When to use this workflow: Parallelization is effective when the divided subtasks can be parallelized for speed, or when multiple perspectives or attempts are needed for higher confidence results. For complex tasks with multiple considerations, LLMs generally perform better when each consideration is handled by a separate LLM call, allowing focused attention on each specific aspect.

并行化有用的示例：

Examples where parallelization is useful:

**分段：**

Sectioning:

- 实现防护措施，其中一个模型实例处理用户查询，而另一个筛选不适当的内容或请求。这往往比让同一个 LLM 调用同时处理防护措施和核心响应表现更好。
- 自动化评估以评估 LLM 性能，其中每次 LLM 调用评估模型在给定提示上的性能的不同方面。

- Implementing guardrails where one model instance processes user queries while another screens them for inappropriate content or requests. This tends to perform better than having the same LLM call handle both guardrails and the core response.
- Automating evals for evaluating LLM performance, where each LLM call evaluates a different aspect of the model's performance on a given prompt.

**投票：**

Voting:

- 审查一段代码的漏洞，其中几个不同的提示审查并标记代码（如果发现问题）。
- 评估给定内容是否不适当，多个提示评估不同方面或需要不同的投票阈值来平衡误报和漏报。

- Reviewing a piece of code for vulnerabilities, where several different prompts review and flag the code if they find a problem.
- Evaluating whether a given piece of content is inappropriate, with multiple prompts evaluating different aspects or requiring different vote thresholds to balance false positives and negatives.

### 工作流：编排者-工作者

Workflow: Orchestrator-workers

在编排者-工作者工作流中，中央 LLM 动态分解任务，将它们委派给工作者 LLM，并综合它们的结果。

In the orchestrator-workers workflow, a central LLM dynamically breaks down tasks, delegates them to worker LLMs, and synthesizes their results.

**编排者-工作者工作流**

The orchestrator-workers workflow

**何时使用此工作流：** 此工作流非常适合无法预测所需子任务的复杂任务（例如，在编码中，需要更改的文件数量和每个文件中更改的性质可能取决于任务）。虽然它在拓扑上相似，但与并行化的关键区别在于其灵活性——子任务不是预定义的，而是由编排者根据特定输入确定的。

When to use this workflow: This workflow is well-suited for complex tasks where you can't predict the subtasks needed (in coding, for example, the number of files that need to be changed and the nature of the change in each file likely depend on the task). Whereas it's topographically similar, the key difference from parallelization is its flexibility—subtasks aren't pre-defined, but determined by the orchestrator based on the specific input.

编排者-工作者有用的示例：

Example where orchestrator-workers is useful:

- 每次对多个文件进行复杂更改的编码产品。
- 涉及从多个来源收集和分析信息以获取可能相关信息的搜索任务。

- Coding products that make complex changes to multiple files each time.
- Search tasks that involve gathering and analyzing information from multiple sources for possible relevant information.

### 工作流：评估者-优化者

Workflow: Evaluator-optimizer

在评估者-优化者工作流中，一次 LLM 调用生成响应，而另一次在循环中提供评估和反馈。

In the evaluator-optimizer workflow, one LLM call generates a response while another provides evaluation and feedback in a loop.

**评估者-优化者工作流**

The evaluator-optimizer workflow

**何时使用此工作流：** 当我们有明确的评估标准，并且迭代改进提供可衡量的价值时，此工作流特别有效。良好契合的两个迹象是，首先，当人类阐述其反馈时，LLM 响应可以得到明显改进；其次，LLM 可以提供这样的反馈。这类似于人类作家在创作精美文档时可能经历的迭代写作过程。

When to use this workflow: This workflow is particularly effective when we have clear evaluation criteria, and when iterative refinement provides measurable value. The two signs of good fit are, first, that LLM responses can be demonstrably improved when a human articulates their feedback; and second, that the LLM can provide such feedback. This is analogous to the iterative writing process a human writer might go through when producing a polished document.

评估者-优化者有用的示例：

Examples where evaluator-optimizer is useful:

- 文学翻译，其中存在翻译者 LLM 最初可能无法捕捉的细微差别，但评估者 LLM 可以提供有用的批评。
- 需要多轮搜索和分析以收集全面信息的复杂搜索任务，其中评估者决定是否需要进一步搜索。

- Literary translation where there are nuances that the translator LLM might not capture initially, but where an evaluator LLM can provide useful critiques.
- Complex search tasks that require multiple rounds of searching and analysis to gather comprehensive information, where the evaluator decides whether further searches are warranted.

### 智能体

Agents

随着 LLM 在关键能力方面的成熟——理解复杂输入、进行推理和规划、可靠地使用工具以及从错误中恢复，智能体正在生产中出现。智能体通过来自人类用户的命令或与人类用户的交互式讨论开始工作。一旦任务明确，智能体就会独立计划和运行，可能会返回给人类以获取更多信息或判断。在执行期间，智能体在每一步从环境中获取"真实情况"（如工具调用结果或代码执行）以评估其进度至关重要。然后，智能体可以在检查点或遇到障碍时暂停以获取人类反馈。任务通常在完成时终止，但通常也包括停止条件（如最大迭代次数）以保持控制。

Agents are emerging in production as LLMs mature in key capabilities—understanding complex inputs, engaging in reasoning and planning, using tools reliably, and recovering from errors. Agents begin their work with either a command from, or interactive discussion with, the human user. Once the task is clear, agents plan and operate independently, potentially returning to the human for further information or judgement. During execution, it's crucial for the agents to gain "ground truth" from the environment at each step (such as tool call results or code execution) to assess its progress. Agents can then pause for human feedback at checkpoints or when encountering blockers. The task often terminates upon completion, but it's also common to include stopping conditions (such as a maximum number of iterations) to maintain control.

智能体可以处理复杂的任务，但它们的实现通常很直接。它们通常只是在循环中基于环境反馈使用工具的 LLM。因此，清晰、周到地设计工具集及其文档至关重要。我们在附录 2（"提示工程你的工具"）中扩展了工具开发的最佳实践。

Agents can handle sophisticated tasks, but their implementation is often straightforward. They are typically just LLMs using tools based on environmental feedback in a loop. It is therefore crucial to design toolsets and their documentation clearly and thoughtfully. We expand on best practices for tool development in Appendix 2 ("Prompt Engineering your Tools").

**自主智能体**

Autonomous agent

**何时使用智能体：** 智能体可用于开放式问题，在这些问题中难以或不可能预测所需的步骤数，并且你无法硬编码固定路径。LLM 将可能运行多个回合，你必须对其决策有一定程度的信任。智能体的自主性使它们非常适合在受信任的环境中扩展任务。

When to use agents: Agents can be used for open-ended problems where it's difficult or impossible to predict the required number of steps, and where you can't hardcode a fixed path. The LLM will potentially operate for many turns, and you must have some level of trust in its decision-making. Agents' autonomy makes them ideal for scaling tasks in trusted environments.

智能体的自主性意味着更高的成本和复合错误的可能性。我们建议在沙盒环境中进行广泛测试，并采取适当的防护措施。

The autonomous nature of agents means higher costs, and the potential for compounding errors. We recommend extensive testing in sandboxed environments, along with the appropriate guardrails.

智能体有用的示例：

Examples where agents are useful:

以下示例来自我们自己的实现：

The following examples are from our own implementations:

- 用于解决 SWE-bench 任务的编码智能体，这些任务涉及根据任务描述对许多文件进行编辑；
- 我们的"计算机使用"参考实现，其中 Claude 使用计算机完成任务。

- A coding Agent to resolve SWE-bench tasks, which involve edits to many files based on a task description;
- Our "computer use" reference implementation, where Claude uses a computer to accomplish tasks.

**编码智能体的高级流程**

High-level flow of a coding agent

## 组合和定制这些模式

Combining and customizing these patterns

这些构建块不是规定性的。它们是开发者可以塑造和组合以适应不同用例的常见模式。与任何 LLM 功能一样，成功的关键是衡量性能并迭代实现。重申一下：你应该只在明显改善结果时才考虑增加复杂性。

These building blocks aren't prescriptive. They're common patterns that developers can shape and combine to fit different use cases. The key to success, as with any LLM features, is measuring performance and iterating on implementations. To repeat: you should consider adding complexity only when it demonstrably improves outcomes.

## 总结

Summary

在 LLM 领域取得成功不在于构建最复杂的系统，而在于为你的需求构建正确的系统。从简单的提示开始，通过全面的评估对其进行优化，只有当更简单的解决方案不足时才添加多步智能体系统。

Success in the LLM space isn't about building the most sophisticated system. It's about building the right system for your needs. Start with simple prompts, optimize them with comprehensive evaluation, and add multi-step agentic systems only when simpler solutions fall short.

在实现智能体时，我们尝试遵循三个核心原则：

When implementing agents, we try to follow three core principles:

- 在智能体设计中保持简单性。
- 通过明确显示智能体的规划步骤来优先考虑透明度。
- 通过全面的工具文档和测试来精心设计你的智能体-计算机接口（ACI）。

- Maintain simplicity in your agent's design.
- Prioritize transparency by explicitly showing the agent's planning steps.
- Carefully craft your agent-computer interface (ACI) through thorough tool documentation and testing.

框架可以帮助你快速入门，但在转向生产时，不要犹豫减少抽象层并使用基本组件构建。通过遵循这些原则，你可以创建不仅强大而且可靠、可维护且受用户信任的智能体。

Frameworks can help you get started quickly, but don't hesitate to reduce abstraction layers and build with basic components as you move to production. By following these principles, you can create agents that are not only powerful but also reliable, maintainable, and trusted by their users.

## 致谢

Acknowledgements

作者：Erik Schluntz 和 Barry Zhang。这项工作借鉴了我们在 Anthropic 构建智能体的经验以及客户分享的宝贵见解，对此我们深表感激。

Written by Erik Schluntz and Barry Zhang. This work draws upon our experiences building agents at Anthropic and the valuable insights shared by our customers, for which we're deeply grateful.

## 附录 1：实践中的智能体

Appendix 1: Agents in practice

我们与客户的合作揭示了 AI 智能体的两个特别有前景的应用，这些应用展示了上述模式的实际价值。这两个应用都说明了智能体为需要对话和行动、具有明确成功标准、启用反馈循环并整合有意义的人工监督的任务增加了最大价值。

Our work with customers has revealed two particularly promising applications for AI agents that demonstrate the practical value of the patterns discussed above. Both applications illustrate how agents add the most value for tasks that require both conversation and action, have clear success criteria, enable feedback loops, and integrate meaningful human oversight.

### A. 客户支持

A. Customer support

客户支持通过工具集成将熟悉的聊天机器人界面与增强的能力相结合。这非常适合更开放式的智能体，因为：

Customer support combines familiar chatbot interfaces with enhanced capabilities through tool integration. This is a natural fit for more open-ended agents because:

- 支持交互自然遵循对话流程，同时需要访问外部信息和操作；
- 可以集成工具来提取客户数据、订单历史和知识库文章；
- 诸如发放退款或更新工单之类的操作可以以程序化方式处理；以及
- 成功可以通过用户定义的解决方案明确衡量。

- Support interactions naturally follow a conversation flow while requiring access to external information and actions;
- Tools can be integrated to pull customer data, order history, and knowledge base articles;
- Actions such as issuing refunds or updating tickets can be handled programmatically; and
- Success can be clearly measured through user-defined resolutions.

几家公司已经通过基于使用量的定价模型展示了这种方法的可行性，这些模型仅对成功的解决方案收费，显示了对其智能体有效性的信心。

Several companies have demonstrated the viability of this approach through usage-based pricing models that charge only for successful resolutions, showing confidence in their agents' effectiveness.

### B. 编码智能体

B. Coding agents

软件开发领域已显示出 LLM 功能的巨大潜力，能力从代码补全演变到自主问题解决。智能体特别有效，因为：

The software development space has shown remarkable potential for LLM features, with capabilities evolving from code completion to autonomous problem-solving. Agents are particularly effective because:

- 代码解决方案可以通过自动化测试进行验证；
- 智能体可以使用测试结果作为反馈来迭代解决方案；
- 问题空间定义明确且结构化；以及
- 输出质量可以客观地衡量。

- Code solutions are verifiable through automated tests;
- Agents can iterate on solutions using test results as feedback;
- The problem space is well-defined and structured; and
- Output quality can be measured objectively.

在我们自己的实现中，智能体现在可以仅基于拉取请求描述来解决 SWE-bench Verified 基准测试中的真实 GitHub 问题。然而，虽然自动化测试有助于验证功能，但人工审查对于确保解决方案与更广泛的系统要求保持一致仍然至关重要。

In our own implementation, agents can now solve real GitHub issues in the SWE-bench Verified benchmark based on the pull request description alone. However, whereas automated testing helps verify functionality, human review remains crucial for ensuring solutions align with broader system requirements.

## 附录 2：提示工程你的工具

Appendix 2: Prompt engineering your tools

无论你正在构建哪种智能体系统，工具都可能是智能体的重要组成部分。工具通过在我们的 API 中指定其确切的结构和定义，使 Claude 能够与外部服务和 API 交互。当 Claude 响应时，如果它计划调用工具，它将在 API 响应中包含工具使用块。工具定义和规范应该得到与整体提示一样多的提示工程关注。在这个简短的附录中，我们描述了如何对你的工具进行提示工程。

No matter which agentic system you're building, tools will likely be an important part of your agent. Tools enable Claude to interact with external services and APIs by specifying their exact structure and definition in our API. When Claude responds, it will include a tool use block in the API response if it plans to invoke a tool. Tool definitions and specifications should be given just as much prompt engineering attention as your overall prompts. In this brief appendix, we describe how to prompt engineer your tools.

通常有几种方法可以指定相同的操作。例如，你可以通过编写差异或重写整个文件来指定文件编辑。对于结构化输出，你可以在 markdown 或 JSON 中返回代码。在软件工程中，这样的差异是表面的，可以无损地从一种转换为另一种。然而，某些格式对于 LLM 来说比其他格式更难编写。编写差异需要在编写新代码之前知道块头中有多少行正在更改。在 JSON 中编写代码（与 markdown 相比）需要额外的换行符和引号转义。

There are often several ways to specify the same action. For instance, you can specify a file edit by writing a diff, or by rewriting the entire file. For structured output, you can return code inside markdown or inside JSON. In software engineering, differences like these are cosmetic and can be converted losslessly from one to the other. However, some formats are much more difficult for an LLM to write than others. Writing a diff requires knowing how many lines are changing in the chunk header before the new code is written. Writing code inside JSON (compared to markdown) requires extra escaping of newlines and quotes.

我们对决定工具格式的建议如下：

Our suggestions for deciding on tool formats are the following:

- 在模型将自己写入困境之前，给它足够的标记来"思考"。
- 使格式接近模型在互联网上的文本中自然看到的内容。
- 确保没有格式"开销"，例如必须保持数千行代码的准确计数，或对其编写的任何代码进行字符串转义。

- Give the model enough tokens to "think" before it writes itself into a corner.
- Keep the format close to what the model has seen naturally occurring in text on the internet.
- Make sure there's no formatting "overhead" such as having to keep an accurate count of thousands of lines of code, or string-escaping any code it writes.

一个经验法则是考虑投入人机接口（HCI）的努力有多大，并计划在创建良好的智能体-计算机接口（ACI）方面投入同样多的努力。以下是一些关于如何做到这一点的想法：

One rule of thumb is to think about how much effort goes into human-computer interfaces (HCI), and plan to invest just as much effort in creating good agent-computer interfaces (ACI). Here are some thoughts on how to do so:

**设身处地为模型着想。** 根据描述和参数，如何使用此工具是否显而易见，还是你需要仔细思考？如果是这样，那么模型可能也是如此。一个好的工具定义通常包括示例用法、边缘情况、输入格式要求以及与其他工具的明确界限。

Put yourself in the model's shoes. Is it obvious how to use this tool, based on the description and parameters, or would you need to think carefully about it? If so, then it's probably also true for the model. A good tool definition often includes example usage, edge cases, input format requirements, and clear boundaries from other tools.

**你如何更改参数名称或描述以使事情更明显？** 将此视为为团队中的初级开发人员编写出色的文档字符串。这在使用许多类似工具时尤为重要。

How can you change parameter names or descriptions to make things more obvious? Think of this as writing a great docstring for a junior developer on your team. This is especially important when using many similar tools.

**测试模型如何使用你的工具：** 在我们的工作台中运行许多示例输入，以查看模型犯了哪些错误，并进行迭代。

Test how the model uses your tools: Run many example inputs in our workbench to see what mistakes the model makes, and iterate.

**对你的工具进行防错设计。** 更改参数，使其更难犯错。

Poka-yoke your tools. Change the arguments so that it is harder to make mistakes.

在为 SWE-bench 构建智能体时，我们实际上花在优化工具上的时间比花在整体提示上的时间更多。例如，我们发现在智能体移出根目录后，模型会在使用相对文件路径的工具中犯错误。为了解决这个问题，我们更改了工具，使其始终需要绝对文件路径——我们发现模型完美地使用了这种方法。

While building our agent for SWE-bench, we actually spent more time optimizing our tools than the overall prompt. For example, we found that the model would make mistakes with tools using relative filepaths after the agent had moved out of the root directory. To fix this, we changed the tool to always require absolute filepaths—and we found that the model used this method flawlessly.
