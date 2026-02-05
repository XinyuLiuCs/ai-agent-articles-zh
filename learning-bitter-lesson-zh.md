# 学习痛苦的教训

*Learning the Bitter Lesson*
https://rlancemartin.github.io/2025/07/30/bitter_lesson/

Lance Martin
2025年7月30日

从70年的人工智能研究中可以得出的最大教训是:利用计算的通用方法最终是最有效的,而且优势极为明显。

From 70 years of AI research: general methods that leverage computation are ultimately the most effective, and by a large margin.

Rich Sutton,《痛苦的教训》

Rich Sutton, The Bitter Lesson

痛苦的教训在AI研究的许多领域中被一次又一次地学习,比如国际象棋、围棋、语音、视觉。事实证明,利用计算是最重要的事情,而我们强加在模型上的"结构"往往限制了它们利用不断增长的计算能力的能力。

The Bitter Lesson has been learned over and over across many domains of AI research, such as Chess, Go, speech, vision. Leveraging computation turns out to be the most important thing and the "structure" we impose on models often limits their ability to leverage growing computation.

我们所说的"结构"是什么意思?结构通常包括关于我们期望模型如何解决问题的归纳偏置。计算机视觉就是一个很好的例子。几十年来,研究人员基于领域知识设计特征(例如SIFT和HOG)。但这些手工制作的特征将模型限制在我们预期的模式中。随着计算和数据的扩展,直接从像素中学习特征的深度网络超越了手工制作的方法。

What do we mean by "structure"? Often structure includes inductive biases about how we expect models to solve problems. Computer vision is a good example. For decades, researchers designed features (e.g., SIFT and HOG) based upon domain knowledge. But these hand-crafted features restricted models to the patterns that we anticipated. As computation and data scaled, deep networks that learned features directly from pixels outperformed hand-crafted methods.

考虑到这一点,Hyung Won Chung(OpenAI)关于他的研究方法的演讲很有意思:

With this in mind, Hyung Won Chung's (OpenAI) talk about his research approach is interesting:

添加给定计算和数据水平所需的结构。之后移除它们,因为这些捷径会成为进一步改进的瓶颈。

Add structures needed for the given level of compute and data available. Remove them later, because these shortcuts will bottleneck further improvement.

## AI工程中的痛苦教训

The Bitter Lesson in AI Engineering

痛苦的教训也适用于AI工程,即在快速改进的模型之上构建应用程序的技艺。举个例子,Boris(Claude Code负责人)提到痛苦的教训强烈影响了他的方法。而且我发现Hyung的演讲为AI工程提供了一些有用的教训。下面我将用构建open-deep-research的故事来说明这一点。

The Bitter Lessons also applies to AI engineering, the craft of building applications on top of rapidly improving models. As an example, Boris (lead on Claude Code) mentioned that the Bitter Lesson strongly influenced his approach. And I've found that Hyung's talk provides some useful lessons for AI engineering. Below I'll illustrate this with a story about building open-deep-research.

### 添加结构

Adding Structure

我在2023年对智能体有过令人沮丧的经历。很难让LLM可靠地调用工具,而且上下文窗口很小。在2024年初,我开始对工作流感兴趣。工作流不是让LLM在循环中自主调用工具,而是将LLM调用嵌入到预定义的代码路径中。

I had frustrating experiences with agents in 2023. It was hard to get LLMs to reliably call tools and context windows were small. In early 2024, I became interested in workflows. Rather than an LLM autonomously calling tools in a loop, workflows embed LLM calls in predefined code paths.

在2024年底,我发布了一个用于网络研究的编排器-工作器工作流。编排器是一个LLM调用,它接受用户的请求并返回要撰写的报告部分列表。一组工作器并行化研究和撰写所有报告部分。然后,我只是将它们组合在一起。

In late 2024, I released an orchestrator-worker workflow for web research. The orchestrator was an LLM call that took a user's request and returned a list of report sections to write. A set of workers parallelized research and writing all report sections. Then, I just combined them together.

那么,什么是"结构"?我强加了一些关于LLM应该如何执行快速、可靠研究的假设:它应该将请求分解为一组报告部分,并行研究和撰写它们以提高速度,并避免工具调用以提高可靠性。

So, what was the "structure"? I imposed a few assumptions about how LLMs should perform fast, reliable research: it should decompose the request into a set of report sections, research and write them in parallel to make it fast, and avoid tool calling to make it more reliable.

### 瓶颈

Bottlenecks

随着2024年接近尾声,事情开始发生变化。工具调用正在变得更好。到2025年冬季,MCP获得了显著的势头,很明显智能体非常适合研究任务。但是,我强加的结构阻止我利用这些改进。

Things began to shift as 2024 came to a close. Tool calling was getting better. By winter 2025, MCP had gained significant momentum and it was clear that agents were well suited to the research task. But, the structure I imposed prevented me from leveraging these improvements.

我没有使用工具调用,所以无法利用不断增长的MCP生态系统。工作流总是将请求分解为报告部分,这是一种僵化的研究策略,并不适用于所有情况。报告有时也是脱节的,因为我强制工作器并行撰写部分。

I did not use tool calling, so I could not take advantage of the growing MCP ecosystem. The workflow always decomposed the request into report sections, a rigid research strategy that was not appropriate for all cases. The reports also were sometimes disjoint because I forced workers to write sections in parallel.

### 移除结构

Removing Structure

我转向了多智能体系统,这让我能够使用工具并让系统灵活地规划研究策略。但我设计时让每个子智能体仍然撰写自己的报告部分。这遇到了Cognition的Walden Yan指出的一个问题:**多智能体系统很难,因为子智能体往往无法有效沟通**。报告仍然是脱节的,因为我的子智能体并行撰写部分。

I moved to a multi-agent system, which allowed me to use tools and let the system flexibly plan the research strategy. But I designed it such that each sub-agent still wrote its own report section. This ran into a problem that Walden Yan of Cognition has called out: multi-agent systems are hard because the sub-agents often don't communicate effectively. Reports were still disjoint because my sub-agents wrote sections in parallel.

这是Hyung演讲的主要观点之一:在我们更新方法时,往往未能移除我们添加的所有结构。在我的情况下,我转向了智能体,但仍然强制每个智能体并行撰写报告的一部分。

This is one of the main points of Hyung's talk: we often fail to remove all the structure we add as we update our methods. In my case, I moved to an agent, but still was forcing each agent to write part of the report in parallel.

我将撰写移到了最后一步。系统现在可以灵活地规划研究策略,使用多智能体上下文收集,并根据收集的上下文一次性撰写报告。它在深度研究基准上得分43.5(前10名),对于一个小型开源项目来说还不错(而且接近使用强化学习或受益于更大规模工作的智能体)。

I moved writing to a final step. The system could now flexibly plan the research strategy, use multi-agent context gathering, and write the report in one-shot based on the collected context. It scores a 43.5 on deep research bench (top 10), which is not bad for a small open source effort (and close to agents that use RL or benefit from much larger-scale efforts).

## 教训

Lessons

AI工程可以从Chung演讲中得出的一些简单教训中受益:

AI engineering can benefit from some simple lessons drawn from Chung's talk:

理解你的应用结构

Understand your application structure

随着模型改进重新评估结构

**Re-evaluate structure as models improve**

使移除结构变得容易

**Make it easy to remove structure**

关于第一点,考虑你的应用程序设计中嵌入了哪些LLM性能假设。对于我最初的工作流,我避免了工具调用,因为(当时)它不可靠。但几个月后这已不再是事实!Jared Kaplan(Anthropic联合创始人)最近指出,构建"还不太奏效"的东西甚至可能是有益的,因为模型会赶上来(而且通常很快)。

On the first point, consider what LLM performance assumptions are baked into the design of your application. For my initial workflow, I avoided tool calling because (at the time) it was not reliable. This was no longer true a few months later! Jared Kaplan (co-founder of Anthropic) recently made the point that it can even be beneficial to "build things that don't quite work" yet because the models will catch up (often quickly).

关于第二点,随着工具调用的改进,我在重新评估我的假设方面有点慢。关于最后一点,我同意Walden(Cognition)和Harrison(LangChain)的观点,即智能体抽象可能带来风险,因为它们可能使移除结构变得更困难。我仍然使用框架(LangGraph)来利用其有用的通用功能(例如检查点),但坚持使用其低级构建块(例如节点和边),我可以轻松地(重新)配置它们。

On the second point, I was a bit slow to re-evaluate my assumptions as tool calling improved. And, on the final point, I agree with Walden (Cognition) and Harrison (LangChain) that agent abstractions can pose risk because they can make it harder remove structure. I still use a framework (LangGraph) for its useful general features (e.g., checkpointing), but stick to its low-level building blocks (e.g., nodes and edges) that I can easily (re-)configure.

构建AI应用程序的设计哲学还处于初级阶段。尽管如此,正如Hyung所说,关注我们能够预测的驱动力是有帮助的:模型会变得好得多。设计AI应用程序以利用这一点可能是最重要的事情。

The design philosophy for building AI applications is in its infancy. Still, as Hyung said, it is helpful to focus on the driving force that we can predict: model will get much better. Designing AI applications to take advantage of this will probably be the most important thing.

## 致谢

Credits

感谢Vadym Barda的初步评估、MCP支持和有益的讨论。感谢Nick Huang在多智能体实现以及Deep Research Bench评估方面的工作。

Thanks to Vadym Barda for initial evals, MCP support, and helpful discussion. Thanks to Nick Huang for work on the multi-agent implementation as well as Deep Research Bench evals.
