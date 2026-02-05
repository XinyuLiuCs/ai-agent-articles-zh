# 长期运行智能体的有效框架

*Effective harnesses for long-running agents*

https://www.anthropic.com/research/effective-harnesses-long-running-agents

发布于2025年11月26日

Published Nov 26, 2025

智能体在跨多个上下文窗口工作时仍面临挑战。我们从人类工程师身上汲取灵感,为长期运行智能体创建了一个更有效的框架。

Agents still face challenges working across many context windows. We looked to human engineers for inspiration in creating a more effective harness for long-running agents.

随着AI智能体变得越来越强大,开发人员越来越多地要求它们承担需要跨越数小时甚至数天工作的复杂任务。然而,让智能体在多个上下文窗口之间持续取得进展仍然是一个未解决的问题。

As AI agents become more capable, developers are increasingly asking them to take on complex tasks requiring work that spans hours, or even days. However, getting agents to make consistent progress across multiple context windows remains an open problem.

长期运行智能体的核心挑战在于它们必须在离散的会话中工作,每个新会话开始时都没有之前发生的事情的记忆。想象一个由轮班工作的工程师组成的软件项目团队,每个新到达的工程师都对上一班发生的事情毫无记忆。由于上下文窗口是有限的,而且大多数复杂项目无法在单个窗口内完成,智能体需要一种方法来弥合编码会话之间的差距。

The core challenge of long-running agents is that they must work in discrete sessions, and each new session begins with no memory of what came before. Imagine a software project staffed by engineers working in shifts, where each new engineer arrives with no memory of what happened on the previous shift. Because context windows are limited, and because most complex projects cannot be completed within a single window, agents need a way to bridge the gap between coding sessions.

我们开发了一个双重解决方案,使Claude Agent SDK能够在多个上下文窗口中有效工作:一个初始化智能体在首次运行时设置环境,以及一个编码智能体,其任务是在每个会话中取得增量进展,同时为下一个会话留下清晰的工作成果。您可以在配套的快速入门中找到代码示例。

We developed a two-fold solution to enable the Claude Agent SDK to work effectively across many context windows: an initializer agent that sets up the environment on the first run, and a coding agent that is tasked with making incremental progress in every session, while leaving clear artifacts for the next session. You can find code examples in the accompanying quickstart.

## 长期运行智能体问题

The long-running agent problem

Claude Agent SDK是一个强大的、通用的智能体框架,擅长编码以及其他需要模型使用工具来收集上下文、规划和执行的任务。它具有上下文管理能力,如压缩,使智能体能够在不耗尽上下文窗口的情况下处理任务。理论上,在这种设置下,智能体应该能够在任意长的时间内继续做有用的工作。

The Claude Agent SDK is a powerful, general-purpose agent harness adept at coding, as well as other tasks that require the model to use tools to gather context, plan, and execute. It has context management capabilities such as compaction, which enables an agent to work on a task without exhausting the context window. Theoretically, given this setup, it should be possible for an agent to continue to do useful work for an arbitrarily long time.

然而,压缩是不够的。即使像Opus 4.5这样的前沿编码模型在Claude Agent SDK上跨多个上下文窗口循环运行,如果只给它一个高层次的提示,比如"构建一个claude.ai的克隆",它也无法构建一个生产质量的网络应用。

However, compaction isn't sufficient. Out of the box, even a frontier coding model like Opus 4.5 running on the Claude Agent SDK in a loop across multiple context windows will fall short of building a production-quality web app if it's only given a high-level prompt, such as "build a clone of claude.ai."

Claude的失败表现为两种模式。首先,智能体倾向于**一次尝试做太多事情**——本质上是试图一次性完成应用。通常,这会导致模型在实现过程中耗尽上下文,使下一个会话不得不从一个功能实现了一半且没有文档记录的状态开始。然后智能体必须猜测发生了什么,并花费大量时间试图让基本应用再次工作。即使有压缩,这种情况也会发生,因为压缩并不总是向下一个智能体传递完全清晰的指令。

Claude's failures manifested in two patterns. First, the agent tended to try to do too much at once—essentially to attempt to one-shot the app. Often, this led to the model running out of context in the middle of its implementation, leaving the next session to start with a feature half-implemented and undocumented. The agent would then have to guess at what had happened, and spend substantial time trying to get the basic app working again. This happens even with compaction, which doesn't always pass perfectly clear instructions to the next agent.

第二种失败模式通常发生在项目后期。**在已经构建了一些功能后,后来的智能体实例会环顾四周,看到已经取得了进展,并宣布工作完成。**

A second failure mode would often occur later in a project. After some features had already been built, a later agent instance would look around, see that progress had been made, and declare the job done.

这将问题分解为两个部分。首先,我们需要设置一个初始环境,为给定提示所需的所有功能奠定基础,这使智能体能够逐步、逐个功能地工作。其次,我们应该提示每个智能体朝着其目标取得增量进展,同时在会话结束时让环境保持在一个干净的状态。"干净状态"是指适合合并到主分支的代码:没有重大错误,代码有序且文档完善,总的来说,开发人员可以轻松开始开发新功能,而无需先清理不相关的混乱。

This decomposes the problem into two parts. First, we need to set up an initial environment that lays the foundation for all the features that a given prompt requires, which sets up the agent to work step-by-step and feature-by-feature. Second, we should prompt each agent to make incremental progress towards its goal while also leaving the environment in a clean state at the end of a session. By "clean state" we mean the kind of code that would be appropriate for merging to a main branch: there are no major bugs, the code is orderly and well-documented, and in general, a developer could easily begin work on a new feature without first having to clean up an unrelated mess.

在内部实验时,我们使用双重解决方案解决了这些问题:

When experimenting internally, we addressed these problems using a two-part solution:

**初始化智能体**:第一个智能体会话使用一个专门的提示,要求模型设置初始环境:一个init.sh脚本、一个记录智能体所做工作的claude-progress.txt文件,以及一个显示添加了哪些文件的初始git提交。

**Initializer agent**: The very first agent session uses a specialized prompt that asks the model to set up the initial environment: an init.sh script, a claude-progress.txt file that keeps a log of what agents have done, and an initial git commit that shows what files were added.

**编码智能体**:每个后续会话要求模型取得增量进展,然后留下结构化的更新。<sup>1</sup>

**Coding agent**: Every subsequent session asks the model to make incremental progress, then leave structured updates.<sup>1</sup>

这里的关键洞察是找到一种方法让智能体在以新的上下文窗口开始时快速理解工作状态,这通过claude-progress.txt文件与git历史一起完成。这些实践的灵感来自于了解有效的软件工程师每天都在做什么。

The key insight here was finding a way for agents to quickly understand the state of work when starting with a fresh context window, which is accomplished with the claude-progress.txt file alongside the git history. Inspiration for these practices came from knowing what effective software engineers do every day.

## 环境管理

Environment management

在更新的Claude 4提示指南中,我们分享了多上下文窗口工作流的一些最佳实践,包括一个使用"第一个上下文窗口的不同提示"的框架结构。这个"不同的提示"要求初始化智能体设置环境,包含未来编码智能体有效工作所需的所有必要上下文。在这里,我们对这种环境的一些关键组件进行更深入的探讨。

In the updated Claude 4 prompting guide, we shared some best practices for multi-context window workflows, including a harness structure that uses "a different prompt for the very first context window." This "different prompt" requests that the initializer agent set up the environment with all the necessary context that future coding agents will need to work effectively. Here, we provide a deeper dive on some of the key components of such an environment.

### 功能列表

Feature list

为了解决智能体一次性完成应用或过早认为项目完成的问题,我们提示初始化智能体编写一个全面的功能需求文件,扩展用户的初始提示。在claude.ai克隆示例中,这意味着超过200个功能,例如"用户可以打开新聊天、输入查询、按回车并看到AI响应"。这些功能最初都被标记为"失败",以便后来的编码智能体能够清楚地了解完整功能是什么样子的。

To address the problem of the agent one-shotting an app or prematurely considering the project complete, we prompted the initializer agent to write a comprehensive file of feature requirements expanding on the user's initial prompt. In the claude.ai clone example, this meant over 200 features, such as "a user can open a new chat, type in a query, press enter, and see an AI response." These features were all initially marked as "failing" so that later coding agents would have a clear outline of what full functionality looked like.

```json
{
    "category": "functional",
    "description": "New chat button creates a fresh conversation",
    "steps": [
      "Navigate to main interface",
      "Click the 'New Chat' button",
      "Verify a new conversation is created",
      "Check that chat area shows welcome state",
      "Verify conversation appears in sidebar"
    ],
    "passes": false
  }
```

我们提示编码智能体仅通过更改passes字段的状态来编辑此文件,并使用强烈措辞的指令,如"删除或编辑测试是不可接受的,因为这可能导致功能缺失或存在错误"。经过一些实验,我们选择使用JSON,因为与Markdown文件相比,模型不太可能不当地更改或覆盖JSON文件。

We prompt coding agents to edit this file only by changing the status of a passes field, and we use strongly-worded instructions like "It is unacceptable to remove or edit tests because this could lead to missing or buggy functionality." After some experimentation, we landed on using JSON for this, as the model is less likely to inappropriately change or overwrite JSON files compared to Markdown files.

### 增量进展

Incremental progress

有了这个初始环境脚手架,下一次迭代的编码智能体被要求一次只处理一个功能。事实证明,这种增量方法对于解决智能体一次做太多事情的倾向至关重要。

Given this initial environment scaffolding, the next iteration of the coding agent was then asked to work on only one feature at a time. This incremental approach turned out to be critical to addressing the agent's tendency to do too much at once.

一旦以增量方式工作,模型在进行代码更改后让环境保持在干净状态仍然至关重要。在我们的实验中,我们发现引发这种行为的最佳方法是要求模型将其进展提交到git,并附带描述性的提交消息,并在进度文件中编写其进展摘要。这使模型能够使用git来还原错误的代码更改并恢复代码库的工作状态。

Once working incrementally, it's still essential that the model leaves the environment in a clean state after making a code change. In our experiments, we found that the best way to elicit this behavior was to ask the model to commit its progress to git with descriptive commit messages and to write summaries of its progress in a progress file. This allowed the model to use git to revert bad code changes and recover working states of the code base.

这些方法还提高了效率,因为它们消除了智能体必须猜测发生了什么并花费时间试图让基本应用再次工作的需要。

These approaches also increased efficiency, as they eliminated the need for an agent to have to guess at what had happened and spend its time trying to get the basic app working again.

### 测试

Testing

我们观察到的最后一个主要失败模式是Claude倾向于在没有进行适当测试的情况下将功能标记为完成。在没有明确提示的情况下,Claude倾向于进行代码更改,甚至使用单元测试或针对开发服务器的curl命令进行测试,但无法识别该功能是否端到端地工作。

One final major failure mode that we observed was Claude's tendency to mark a feature as complete without proper testing. Absent explicit prompting, Claude tended to make code changes, and even do testing with unit tests or curl commands against a development server, but would fail recognize that the feature didn't work end-to-end.

在构建网络应用的情况下,一旦明确提示使用浏览器自动化工具并像人类用户那样进行所有测试,Claude在端到端验证功能方面表现得相当好。

In the case of building a web app, Claude mostly did well at verifying features end-to-end once explicitly prompted to use browser automation tools and do all testing as a human user would.

![Claude通过Puppeteer MCP服务器在测试claude.ai克隆时拍摄的截图](Screenshots taken by Claude through the Puppeteer MCP server as it tested the claude.ai clone.)

Screenshots taken by Claude through the Puppeteer MCP server as it tested the claude.ai clone.

为Claude提供这些类型的测试工具显著提高了性能,因为智能体能够识别和修复仅从代码中不明显的错误。

Providing Claude with these kinds of testing tools dramatically improved performance, as the agent was able to identify and fix bugs that weren't obvious from the code alone.

一些问题仍然存在,比如Claude的视觉能力和浏览器自动化工具的限制使得识别每种类型的错误都很困难。例如,Claude无法通过Puppeteer MCP看到浏览器原生的警告模态框,因此依赖这些模态框的功能往往更容易出错。

Some issues remain, like limitations to Claude's vision and to browser automation tools making it difficult to identify every kind of bug. For example, Claude can't see browser-native alert modals through the Puppeteer MCP, and features relying on these modals tended to be buggier as a result.

### 快速上手

Getting up to speed

有了上述所有内容,每个编码智能体都被提示运行一系列步骤来了解情况,有些步骤非常基础但仍然有用:

With all of the above in place, every coding agent is prompted to run through a series of steps to get its bearings, some quite basic but still helpful:

运行pwd查看您正在工作的目录。您只能编辑此目录中的文件。

Run pwd to see the directory you're working in. You'll only be able to edit files in this directory.

读取git日志和进度文件以了解最近的工作内容。

Read the git logs and progress files to get up to speed on what was recently worked on.

读取功能列表文件并选择尚未完成的最高优先级功能进行工作。

Read the features list file and choose the highest-priority feature that's not yet done to work on.

这种方法为Claude在每个会话中节省了一些token,因为它不必弄清楚如何测试代码。要求初始化智能体编写一个可以运行开发服务器的init.sh脚本,然后在实现新功能之前运行基本的端到端测试也很有帮助。

This approach saves Claude some tokens in every session since it doesn't have to figure out how to test the code. It also helps to ask the initializer agent to write an init.sh script that can run the development server, and then run through a basic end-to-end test before implementing a new feature.

在claude.ai克隆的情况下,这意味着智能体总是启动本地开发服务器,并使用Puppeteer MCP启动新聊天、发送消息并接收响应。这确保了Claude可以快速识别应用是否处于损坏状态,并立即修复任何现有的错误。如果智能体改为开始实现新功能,它可能会使问题变得更糟。

In the case of the claude.ai clone, this meant that the agent always started the local development server and used the Puppeteer MCP to start a new chat, send a message, and receive a response. This ensured that Claude could quickly identify if the app had been left in a broken state, and immediately fix any existing bugs. If the agent had instead started implementing a new feature, it would likely make the problem worse.

鉴于所有这些,一个典型的会话以以下助手消息开始:

Given all this, a typical session starts off with the following assistant messages:

```
[Assistant] I'll start by getting my bearings and understanding the current state of the project.
[Tool Use] <bash - pwd>
[Tool Use] <read - claude-progress.txt>
[Tool Use] <read - feature_list.json>
[Assistant] Let me check the git log to see recent work.
[Tool Use] <bash - git log --oneline -20>
[Assistant] Now let me check if there's an init.sh script to restart the servers.
<Starts the development server>
[Assistant] Excellent! Now let me navigate to the application and verify that some fundamental features are still working.
<Tests basic functionality>
[Assistant] Based on my verification testing, I can see that the fundamental functionality is working well. The core chat features, theme switching, conversation loading, and error handling are all functioning correctly. Now let me review the tests.json file more comprehensively to understand what needs to be implemented next.
<Starts work on a new feature>
```

### 智能体失败模式和解决方案

Agent failure modes and solutions

| 问题 | 初始化智能体行为 | 编码智能体行为 |
|------|-----------------|---------------|
| Claude过早地宣布整个项目完成。 | 设置功能列表文件:基于输入规范,设置一个包含端到端功能描述列表的结构化JSON文件。 | 在会话开始时读取功能列表文件。选择一个单独的功能开始工作。 |
| Claude让环境处于有错误或进展未记录的状态。 | 编写初始git仓库和进度说明文件。 | 通过读取进度说明文件和git提交日志开始会话,并在开发服务器上运行基本测试以捕获任何未记录的错误。通过编写git提交和进度更新结束会话。 |
| Claude过早地将功能标记为完成。 | 设置功能列表文件。 | 自我验证所有功能。只有在仔细测试后才将功能标记为"通过"。 |
| Claude必须花时间弄清楚如何运行应用。 | 编写一个可以运行开发服务器的init.sh脚本。 | 通过读取init.sh开始会话。 |

总结长期运行AI智能体中的四种常见失败模式和解决方案。

Summarizing four common failure modes and solutions in long-running AI agents.

## 未来工作

Future work

这项研究展示了长期运行智能体框架中一组可能的解决方案,使模型能够在多个上下文窗口中取得增量进展。然而,仍然存在未解决的问题。

This research demonstrates one possible set of solutions in a long-running agent harness to enable the model to make incremental progress across many context windows. However, there remain open questions.

最值得注意的是,目前尚不清楚单一的通用编码智能体在各种上下文中表现最佳,还是可以通过多智能体架构实现更好的性能。似乎合理的是,专门的智能体,如测试智能体、质量保证智能体或代码清理智能体,可以在软件开发生命周期的子任务中做得更好。

Most notably, it's still unclear whether a single, general-purpose coding agent performs best across contexts, or if better performance can be achieved through a multi-agent architecture. It seems reasonable that specialized agents like a testing agent, a quality assurance agent, or a code cleanup agent, could do an even better job at sub-tasks across the software development lifecycle.

此外,这个演示针对全栈网络应用开发进行了优化。未来的方向是将这些发现推广到其他领域。这些教训中的部分或全部很可能可以应用于例如科学研究或金融建模所需的长期运行智能体任务类型。

Additionally, this demo is optimized for full-stack web app development. A future direction is to generalize these findings to other fields. It's likely that some or all of these lessons can be applied to the types of long-running agentic tasks required in, for example, scientific research or financial modeling.

## 致谢

Acknowledgements

作者:Justin Young。特别感谢David Hershey、Prithvi Rajasakeran、Jeremy Hadfield、Naia Bouscal、Michael Tingley、Jesse Mu、Jake Eaton、Marius Buleandara、Maggie Vo、Pedram Navid、Nadine Yasser和Alex Notov的贡献。

Written by Justin Young. Special thanks to David Hershey, Prithvi Rajasakeran, Jeremy Hadfield, Naia Bouscal, Michael Tingley, Jesse Mu, Jake Eaton, Marius Buleandara, Maggie Vo, Pedram Navid, Nadine Yasser, and Alex Notov for their contributions.

这项工作反映了Anthropic多个团队的集体努力,他们使Claude能够安全地进行长期自主软件工程,特别是代码强化学习和Claude Code团队。欢迎有兴趣做出贡献的候选人在anthropic.com/careers申请。

This work reflects the collective efforts of several teams across Anthropic who made it possible for Claude to safely do long-horizon autonomous software engineering, especially the code RL & Claude Code teams. Interested candidates who would like to contribute are welcome to apply at anthropic.com/careers.

## 脚注

Footnotes

1. 我们在此上下文中将这些称为单独的智能体,仅因为它们有不同的初始用户提示。系统提示、工具集和整体智能体框架在其他方面是相同的。

1. We refer to these as separate agents in this context only because they have different initial user prompts. The system prompt, set of tools, and overall agent harness was otherwise identical.
