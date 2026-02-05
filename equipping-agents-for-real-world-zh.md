# 用 Agent Skills 为真实世界装备智能体

Equipping agents for the real world with Agent Skills---
https://claude.com/blog/equipping-agents-for-the-real-world-with-agent-skills

Claude 很强大，但真正的工作需要过程性知识和组织上下文。我们推出 Agent Skills，这是一种使用文件和文件夹构建专业智能体的新方法。

Claude is powerful, but real work requires procedural knowledge and organizational context. Introducing Agent Skills, a new way to build specialized agents using files and folders.


更新：我们已将 Agent Skills 作为开放标准发布，以实现跨平台可移植性。（2025年12月18日）

Update: We've published Agent Skills as an open standard for cross-platform portability. (December 18, 2025)

随着模型能力的提高，我们现在可以构建与完整计算环境交互的通用智能体。例如，Claude Code 可以使用本地代码执行和文件系统完成跨领域的复杂任务。但随着这些智能体变得更加强大，我们需要更可组合、可扩展和可移植的方式来为它们提供特定领域的专业知识。

As model capabilities improve, we can now build general-purpose agents that interact with full-fledged computing environments. Claude Code, for example, can accomplish complex tasks across domains using local code execution and filesystems. But as these agents become more powerful, we need more composable, scalable, and portable ways to equip them with domain-specific expertise.

这促使我们创建了 **Agent Skills：由指令、脚本和资源组成的有组织文件夹，智能体可以动态发现和加载，以在特定任务上表现更好。**技能通过将你的专业知识打包成 Claude 的可组合资源来扩展 Claude 的能力，将通用智能体转变为适合你需求的专业智能体。

This led us to create Agent Skills: organized folders of instructions, scripts, and resources that agents can discover and load dynamically to perform better at specific tasks. Skills extend Claude's capabilities by packaging your expertise into composable resources for Claude, transforming general-purpose agents into specialized agents that fit your needs.

为智能体构建技能就像为新员工准备入职指南。现在，任何人都可以通过捕获和共享他们的过程性知识，用可组合的能力来专业化他们的智能体，而不是为每个用例构建碎片化的、定制设计的智能体。在本文中，我们将解释什么是 Skills，展示它们如何工作，并分享构建你自己的技能的最佳实践。

Building a skill for an agent is like putting together an onboarding guide for a new hire. Instead of building fragmented, custom-designed agents for each use case, anyone can now specialize their agents with composable capabilities by capturing and sharing their procedural knowledge. In this article, we explain what Skills are, show how they work, and share best practices for building your own.

要激活技能，你只需要为你的智能体编写一个包含自定义指导的 SKILL.md 文件。

To activate skills, all you need to do is write a SKILL.md file with custom guidance for your agent.

技能是一个包含 SKILL.md 文件的目录，该文件包含有组织的指令、脚本和资源文件夹，为智能体提供额外的能力。

A skill is a directory containing a SKILL.md file that contains organized folders of instructions, scripts, and resources that give agents additional capabilities.

## 技能的构成

The anatomy of a skill

要查看 Skills 的实际应用，让我们通过一个真实示例：支持 Claude 最近推出的文档编辑能力的技能之一。Claude 已经对理解 PDF 有很多了解，但在直接操作它们（例如填写表单）方面能力有限。这个 PDF 技能让我们赋予 Claude 这些新能力。

To see Skills in action, let's walk through a real example: one of the skills that powers Claude's recently launched document editing abilities. Claude already knows a lot about understanding PDFs, but is limited in its ability to manipulate them directly (e.g. to fill out a form). This PDF skill lets us give Claude these new abilities.

最简单的情况下，技能是一个包含 SKILL.md 文件的目录。此文件必须以包含一些必需元数据的 YAML 前置元数据开始：名称和描述。在启动时，智能体会将每个已安装技能的名称和描述预加载到其系统提示中。

At its simplest, a skill is a directory that contains a SKILL.md file. This file must start with YAML frontmatter that contains some required metadata: name and description. At startup, the agent pre-loads the name and description of every installed skill into its system prompt.

这个元数据是渐进式披露的第一层：它提供足够的信息让 Claude 知道何时应该使用每个技能，而无需将所有内容加载到上下文中。这个文件的实际正文是第二层细节。如果 Claude 认为该技能与当前任务相关，它将通过将其完整的 SKILL.md 读入上下文来加载该技能。

This metadata is the first level of progressive disclosure: it provides just enough information for Claude to know when each skill should be used without loading all of it into context. The actual body of this file is the second level of detail. If Claude thinks the skill is relevant to the current task, it will load the skill by reading its full SKILL.md into context.

**SKILL.md 文件的构成，包括相关元数据：名称、描述和与技能应采取的特定操作相关的上下文。**

Anatomy of a SKILL.md file including the relevant metadata: name, description, and context related to the specific actions the skill should take.

SKILL.md 文件必须以包含文件名和描述的 YAML 前置元数据开始，这些元数据在启动时加载到其系统提示中。

A SKILL.md file must begin with YAML Frontmatter that contains a file name and description, which is loaded into its system prompt at startup.

随着技能复杂性的增长，它们可能包含太多上下文而无法放入单个 SKILL.md，或者只在特定场景中相关的上下文。在这些情况下，技能可以在技能目录中捆绑其他文件，并从 SKILL.md 中按名称引用它们。这些额外的链接文件是第三层（及更多层）细节，Claude 可以根据需要选择导航和发现。

As skills grow in complexity, they may contain too much context to fit into a single SKILL.md, or context that's relevant only in specific scenarios. In these cases, skills can bundle additional files within the skill directory and reference them by name from SKILL.md. These additional linked files are the third level (and beyond) of detail, which Claude can choose to navigate and discover only as needed.

在下面显示的 PDF 技能中，SKILL.md 引用了技能作者选择与核心 SKILL.md 一起捆绑的两个额外文件（reference.md 和 forms.md）。通过将表单填写指令移动到单独的文件（forms.md），技能作者能够保持技能核心的精简，相信 Claude 只会在填写表单时读取 forms.md。

In the PDF skill shown below, the SKILL.md refers to two additional files (reference.md and forms.md) that the skill author chooses to bundle alongside the core SKILL.md. By moving the form-filling instructions to a separate file (forms.md), the skill author is able to keep the core of the skill lean, trusting that Claude will read forms.md only when filling out a form.

**如何将额外内容捆绑到 SKILL.md 文件中。**

How to bundle additional content into a SKILL.md file.

你可以将更多上下文（通过额外文件）纳入你的技能，然后 Claude 可以根据系统提示触发这些内容。

You can incorporate more context (via additional files) into your skill that can then be triggered by Claude based on the system prompt.

渐进式披露是使 Agent Skills 灵活且可扩展的核心设计原则。就像一本组织良好的手册从目录开始，然后是特定章节，最后是详细附录，技能让 Claude 只在需要时加载信息：

Progressive disclosure is the core design principle that makes Agent Skills flexible and scalable. Like a well-organized manual that starts with a table of contents, then specific chapters, and finally a detailed appendix, skills let Claude load information only as needed:

**此图描述了 Skills 中上下文的渐进式披露。**

This image depicts how progressive disclosure of context in Skills.

具有文件系统和代码执行工具的智能体在处理特定任务时不需要将技能的全部内容读入其上下文窗口。这意味着可以捆绑到技能中的上下文量实际上是无限的。

Agents with a filesystem and code execution tools don't need to read the entirety of a skill into their context window when working on a particular task. This means that the amount of context that can be bundled into a skill is effectively unbounded.

## 技能和上下文窗口

Skills and the context window

下图显示了当技能被用户消息触发时上下文窗口如何变化。

The following diagram shows how the context window changes when a skill is triggered by a user's message.

**此图描述了技能如何在上下文窗口中触发。**

This image depicts how skills are triggered in your context window.

技能通过系统提示在上下文窗口中触发。

Skills are triggered in the context window via your system prompt.

显示的操作序列：

The sequence of operations shown:

首先，上下文窗口具有核心系统提示和每个已安装技能的元数据，以及用户的初始消息；

To start, the context window has the core system prompt and the metadata for each of the installed skills, along with the user's initial message;

Claude 通过调用 Bash 工具读取 pdf/SKILL.md 的内容来触发 PDF 技能；

Claude triggers the PDF skill by invoking a Bash tool to read the contents of pdf/SKILL.md;

Claude 选择读取与技能捆绑在一起的 forms.md 文件；

Claude chooses to read the forms.md file bundled with the skill;

最后，Claude 在从 PDF 技能加载了相关指令后继续执行用户的任务。

Finally, Claude proceeds with the user's task now that it has loaded relevant instructions from the PDF skill.

## 技能和代码执行

Skills and code execution

技能还可以包含 Claude 可自行决定作为工具执行的代码。

Skills can also include code for Claude to execute as tools at its discretion.

大语言模型在许多任务上表现出色，但某些操作更适合传统代码执行。例如，通过标记生成对列表进行排序比简单运行排序算法要昂贵得多。除了效率问题之外，许多应用程序需要只有代码才能提供的确定性可靠性。

Large language models excel at many tasks, but certain operations are better suited for traditional code execution. For example, sorting a list via token generation is far more expensive than simply running a sorting algorithm. Beyond efficiency concerns, many applications require the deterministic reliability that only code can provide.

在我们的示例中，PDF 技能包括一个预先编写的 Python 脚本，该脚本读取 PDF 并提取所有表单字段。Claude 可以运行此脚本，而无需将脚本或 PDF 加载到上下文中。而且因为代码是确定性的，这个工作流是一致且可重复的。

In our example, the PDF skill includes a pre-written Python script that reads a PDF and extracts all form fields. Claude can run this script without loading either the script or the PDF into context. And because code is deterministic, this workflow is consistent and repeatable.

**此图描述了如何通过 Skills 执行代码。**

This image depicts how code is executed via Skills.

技能还可以包含 Claude 根据任务性质自行决定作为工具执行的代码。

Skills can also include code for Claude to execute as tools at its discretion based on the nature of the task.

## 开发和评估技能

Developing and evaluating skills

以下是开始编写和测试技能的一些有用指南：

Here are some helpful guidelines for getting started with authoring and testing skills:

**从评估开始**：通过在代表性任务上运行智能体并观察它们在哪些方面遇到困难或需要额外上下文，来识别智能体能力中的特定差距。然后逐步构建技能以解决这些不足。

Start with evaluation: Identify specific gaps in your agents' capabilities by running them on representative tasks and observing where they struggle or require additional context. Then build skills incrementally to address these shortcomings.

**结构化以实现规模化**：当 SKILL.md 文件变得笨重时，将其内容拆分为单独的文件并引用它们。如果某些上下文是互斥的或很少一起使用，保持路径分离将减少标记使用量。最后，代码既可以作为可执行工具，也可以作为文档。应该清楚 Claude 应该直接运行脚本还是将它们作为参考读入上下文。

Structure for scale: When the SKILL.md file becomes unwieldy, split its content into separate files and reference them. If certain contexts are mutually exclusive or rarely used together, keeping the paths separate will reduce the token usage. Finally, code can serve as both executable tools and as documentation. It should be clear whether Claude should run scripts directly or read them into context as reference.

**从 Claude 的角度思考**：监控 Claude 在实际场景中如何使用你的技能，并根据观察进行迭代：注意意外的轨迹或对某些上下文的过度依赖。特别关注技能的名称和描述。Claude 在决定是否响应当前任务触发技能时会使用这些。

Think from Claude's perspective: Monitor how Claude uses your skill in real scenarios and iterate based on observations: watch for unexpected trajectories or overreliance on certain contexts. Pay special attention to the name and description of your skill. Claude will use these when deciding whether to trigger the skill in response to its current task.

**与 Claude 一起迭代**：当你与 Claude 一起处理任务时，要求 Claude 将其成功的方法和常见错误捕获到技能中的可重用上下文和代码中。如果它在使用技能完成任务时偏离轨道，请它自我反思出了什么问题。这个过程将帮助你发现 Claude 实际需要什么上下文，而不是试图预先预测它。

Iterate with Claude: As you work on a task with Claude, ask Claude to capture its successful approaches and common mistakes into reusable context and code within a skill. If it goes off track when using a skill to complete a task, ask it to self-reflect on what went wrong. This process will help you discover what context Claude actually needs, instead of trying to anticipate it upfront.

## 使用 Skills 时的安全考虑

Security considerations when using Skills

技能通过指令和代码为 Claude 提供新能力。虽然这使它们变得强大，但这也意味着恶意技能可能会在使用它们的环境中引入漏洞，或指示 Claude 外泄数据并采取非预期行动。

Skills provide Claude with new capabilities through instructions and code. While this makes them powerful, it also means that malicious skills may introduce vulnerabilities in the environment where they're used or direct Claude to exfiltrate data and take unintended actions.

我们建议仅从可信来源安装技能。从不太可信的来源安装技能时，请在使用前彻底审核它。首先阅读技能中捆绑的文件内容以了解它的作用，特别关注代码依赖项和捆绑资源，如图像或脚本。同样，注意技能中指示 Claude 连接到可能不受信任的外部网络源的指令或代码。

We recommend installing skills only from trusted sources. When installing a skill from a less-trusted source, thoroughly audit it before use. Start by reading the contents of the files bundled in the skill to understand what it does, paying particular attention to code dependencies and bundled resources like images or scripts. Similarly, pay attention to instructions or code within the skill that instruct Claude to connect to potentially untrusted external network sources.

## Skills 的未来

The future of Skills

Agent Skills 目前在 Claude.ai、Claude Code、Claude Agent SDK 和 Claude 开发者平台上得到支持。

Agent Skills are supported today across Claude.ai, Claude Code, the Claude Agent SDK, and the Claude Developer Platform.

在接下来的几周内，我们将继续添加支持创建、编辑、发现、共享和使用 Skills 的完整生命周期的功能。我们特别期待 Skills 能够帮助组织和个人与 Claude 分享他们的上下文和工作流的机会。我们还将探索 Skills 如何通过教授智能体涉及外部工具和软件的更复杂工作流来补充模型上下文协议（MCP）服务器。

In the coming weeks, we'll continue to add features that support the full lifecycle of creating, editing, discovering, sharing, and using Skills. We're especially excited about the opportunity for Skills to help organizations and individuals share their context and workflows with Claude. We'll also explore how Skills can complement Model Context Protocol (MCP) servers by teaching agents more complex workflows that involve external tools and software.

展望更远的未来，我们希望使智能体能够自己创建、编辑和评估 Skills，让它们将自己的行为模式编码为可重用的能力。

Looking further ahead, we hope to enable agents to create, edit, and evaluate Skills on their own, letting them codify their own patterns of behavior into reusable capabilities.

Skills 是一个简单的概念，具有相应简单的格式。这种简单性使组织、开发者和最终用户更容易构建定制智能体并赋予它们新能力。

Skills are a simple concept with a correspondingly simple format. This simplicity makes it easier for organizations, developers, and end users to build customized agents and give them new capabilities.

我们很期待看到人们用 Skills 构建什么。今天就通过查看我们的 Skills 文档和 cookbook 开始使用吧。

We're excited to see what people build with Skills. Get started today by checking out our Skills docs and cookbook.

## 致谢

Acknowledgements

本文由 Barry Zhang、Keith Lazuka 和 Mahesh Murag 撰写，他们都非常喜欢文件夹。特别感谢 Anthropic 的许多其他人，他们支持、倡导和构建了 Skills。

Written by Barry Zhang, Keith Lazuka, and Mahesh Murag, who all really like folders. Special thanks to the many others across Anthropic who championed, supported, and built Skills.
