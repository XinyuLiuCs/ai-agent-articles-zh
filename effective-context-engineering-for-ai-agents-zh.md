# AI 智能体的有效上下文工程

Effective context engineering for AI agents

发布日期：2025年9月29日

Published Sep 29, 2025

上下文是 AI 智能体的关键资源，但同时也是有限的资源。本文将探讨有效策划和管理驱动智能体运行的上下文的策略。

Context is a critical but finite resource for AI agents. In this post, we explore strategies for effectively curating and managing the context that powers them.

在应用 AI 领域，提示工程曾是关注焦点，经过几年发展后，一个新术语开始崭露头角：上下文工程。使用语言模型进行构建，越来越不再是为提示寻找合适的词语和短语，而是更多地回答这个更广泛的问题："什么样的上下文配置最有可能产生我们期望模型表现出的行为？"

After a few years of prompt engineering being the focus of attention in applied AI, a new term has come to prominence: context engineering. Building with language models is becoming less about finding the right words and phrases for your prompts, and more about answering the broader question of "what configuration of context is most likely to generate our model's desired behavior?"

上下文是指从大语言模型（LLM）采样时包含的标记集合。当前的工程问题是在 LLM 固有约束下优化这些标记的效用，以持续实现期望的结果。有效驾驭 LLM 通常需要以上下文思维方式进行思考——换句话说：考虑 LLM 在任何给定时刻可用的整体状态，以及该状态可能产生哪些潜在行为。

Context refers to the set of tokens included when sampling from a large-language model (LLM). The engineering problem at hand is optimizing the utility of those tokens against the inherent constraints of LLMs in order to consistently achieve a desired outcome. Effectively wrangling LLMs often requires thinking in context — in other words: considering the holistic state available to the LLM at any given time and what potential behaviors that state might yield.

本文将探讨新兴的上下文工程技艺，并为构建可控、有效的智能体提供精炼的心智模型。

In this post, we'll explore the emerging art of context engineering and offer a refined mental model for building steerable, effective agents.

## 上下文工程与提示工程的区别

Context engineering vs. prompt engineering

在 Anthropic，我们将上下文工程视为提示工程的自然演进。提示工程是指为获得最佳结果而编写和组织 LLM 指令的方法（请参阅我们的文档以了解概述和有用的提示工程策略）。上下文工程则是指在 LLM 推理期间策划和维护最优标记集合（信息）的一系列策略，包括提示之外可能进入上下文的所有其他信息。

At Anthropic, we view context engineering as the natural progression of prompt engineering. Prompt engineering refers to methods for writing and organizing LLM instructions for optimal outcomes (see our docs for an overview and useful prompt engineering strategies). Context engineering refers to the set of strategies for curating and maintaining the optimal set of tokens (information) during LLM inference, including all the other information that may land there outside of the prompts.

在 LLM 工程的早期阶段，提示是 AI 工程工作的最大组成部分，因为日常聊天交互之外的大多数用例都需要针对一次性分类或文本生成任务优化的提示。正如该术语所暗示的，提示工程的主要关注点是如何编写有效的提示，特别是系统提示。然而，随着我们转向构建更强大的智能体——它们能够在多轮推理和更长时间跨度上运行，我们需要管理整个上下文状态的策略（系统指令、工具、模型上下文协议（MCP）、外部数据、消息历史等）。

In the early days of engineering with LLMs, prompting was the biggest component of AI engineering work, as the majority of use cases outside of everyday chat interactions required prompts optimized for one-shot classification or text generation tasks. As the term implies, the primary focus of prompt engineering is how to write effective prompts, particularly system prompts. However, as we move towards engineering more capable agents that operate over multiple turns of inference and longer time horizons, we need strategies for managing the entire context state (system instructions, tools, Model Context Protocol (MCP), external data, message history, etc).

在循环中运行的智能体会生成越来越多可能与下一轮推理相关的数据，这些信息必须循环精炼。上下文工程是从不断演变的可能信息宇宙中策划将进入有限上下文窗口内容的艺术和科学。

An agent running in a loop generates more and more data that could be relevant for the next turn of inference, and this information must be cyclically refined. Context engineering is the art and science of curating what will go into the limited context window from that constantly evolving universe of possible information.

**提示工程与上下文工程**

Prompt engineering vs. context engineering

与编写提示这种离散任务相比，上下文工程是迭代的，每次我们决定传递什么给模型时都会进行策划阶段。

In contrast to the discrete task of writing a prompt, context engineering is iterative and the curation phase happens each time we decide what to pass to the model.

## 为何上下文工程对构建强大智能体至关重要

Why context engineering is important to building capable agents

尽管 LLM 处理速度快且能够管理越来越大量的数据，我们观察到 LLM 和人类一样，在某个点上会失去焦点或经历混淆。针对大海捞针式基准测试的研究揭示了**上下文腐化的概念：随着上下文窗口中标记数量的增加，模型从该上下文中准确回忆信息的能力会下降。**

Despite their speed and ability to manage larger and larger volumes of data, we've observed that LLMs, like humans, lose focus or experience confusion at a certain point. Studies on needle-in-a-haystack style benchmarking have uncovered the concept of context rot: as the number of tokens in the context window increases, the model's ability to accurately recall information from that context decreases.

虽然某些模型表现出更温和的退化，但这一特征在所有模型中都会出现。因此，上下文必须被视为边际收益递减的有限资源。就像人类的工作记忆容量有限一样，LLM 拥有"注意力预算"，在解析大量上下文时会消耗这一预算。引入的每个新标记都会在一定程度上耗尽这个预算，这增加了仔细策划 LLM 可用标记的必要性。

While some models exhibit more gentle degradation than others, this characteristic emerges across all models. Context, therefore, must be treated as a finite resource with diminishing marginal returns. Like humans, who have limited working memory capacity, LLMs have an "attention budget" that they draw on when parsing large volumes of context. Every new token introduced depletes this budget by some amount, increasing the need to carefully curate the tokens available to the LLM.

这种注意力稀缺源于 LLM 的架构约束。**LLM 基于 Transformer 架构，该架构使每个标记能够关注整个上下文中的每个其他标记。这导致 n 个标记之间产生 n² 个成对关系。**

This attention scarcity stems from architectural constraints of LLMs. LLMs are based on the transformer architecture, which enables every token to attend to every other token across the entire context. This results in n² pairwise relationships for n tokens.

随着上下文长度的增加，模型捕获这些成对关系的能力被拉伸变薄，在上下文大小和注意力聚焦之间产生自然张力。此外，**模型从训练数据分布中发展其注意力模式，其中较短序列通常比较长序列更常见。**这意味着模型对上下文范围依赖关系的经验较少，专门参数也较少。

As its context length increases, a model's ability to capture these pairwise relationships gets stretched thin, creating a natural tension between context size and attention focus. Additionally, models develop their attention patterns from training data distributions where shorter sequences are typically more common than longer ones. This means models have less experience with, and fewer specialized parameters for, context-wide dependencies.

位置编码插值等技术允许模型通过将其适配到原始训练的较小上下文来处理更长序列，尽管在标记位置理解上有一些退化。这些因素创造了性能梯度而非硬性断崖：模型在更长上下文中仍然保持高度能力，但与在较短上下文上的表现相比，信息检索和长距离推理的精度可能有所降低。

Techniques like position encoding interpolation allow models to handle longer sequences by adapting them to the originally trained smaller context, though with some degradation in token position understanding. These factors create a performance gradient rather than a hard cliff: models remain highly capable at longer contexts but may show reduced precision for information retrieval and long-range reasoning compared to their performance on shorter contexts.

这些现实意味着深思熟虑的上下文工程对构建强大智能体至关重要。

These realities mean that thoughtful context engineering is essential for building capable agents.

## 有效上下文的构成

The anatomy of effective context

鉴于 LLM 受到有限注意力预算的约束，良好的上下文工程意味着找到最小可能的高信号标记集，以最大化某些期望结果的可能性。实施这一实践说起来容易做起来难，但在以下部分中，我们概述了这一指导原则在上下文的不同组件中的实际意义。

Given that LLMs are constrained by a finite attention budget, good context engineering means finding the smallest possible set of high-signal tokens that maximize the likelihood of some desired outcome. Implementing this practice is much easier said than done, but in the following section, we outline what this guiding principle means in practice across the different components of context.

**系统提示应该极其清晰，使用简单、直接的语言，在适合智能体的高度上呈现想法。**合适的高度是介于两种常见失败模式之间的黄金地带。在一个极端，我们看到工程师在提示中硬编码复杂、脆弱的逻辑来引出精确的智能体行为。这种方法会产生脆弱性，并随着时间推移增加维护复杂性。在另一个极端，工程师有时提供模糊的高层指导，未能给 LLM 提供期望输出的具体信号，或错误地假设了共享上下文。最佳高度在两者之间取得平衡：**足够具体以有效指导行为，但又足够灵活以为模型提供强大的启发式规则来指导行为。**

System prompts should be extremely clear and use simple, direct language that presents ideas at the right altitude for the agent. The right altitude is the Goldilocks zone between two common failure modes. At one extreme, we see engineers hardcoding complex, brittle logic in their prompts to elicit exact agentic behavior. This approach creates fragility and increases maintenance complexity over time. At the other extreme, engineers sometimes provide vague, high-level guidance that fails to give the LLM concrete signals for desired outputs or falsely assumes shared context. The optimal altitude strikes a balance: specific enough to guide behavior effectively, yet flexible enough to provide the model with strong heuristics to guide behavior.

**在上下文工程过程中校准系统提示。**

Calibrating the system prompt in the process of context engineering.

在光谱的一端，我们看到脆弱的硬编码 if-else 提示，在另一端我们看到过于笼统或错误假设共享上下文的提示。

At one end of the spectrum, we see brittle if-else hardcoded prompts, and at the other end we see prompts that are overly general or falsely assume shared context.

我们建议将提示组织成不同的部分（如 `<background_information>`、`<instructions>`、`## Tool guidance`、`## Output description` 等），并使用 XML 标签或 Markdown 标题等技术来划分这些部分，尽管随着模型变得更强大，提示的确切格式可能变得不那么重要。

We recommend organizing prompts into distinct sections (like <background_information>, <instructions>, ## Tool guidance, ## Output description, etc) and using techniques like XML tagging or Markdown headers to delineate these sections, although the exact formatting of prompts is likely becoming less important as models become more capable.

无论你决定如何构建系统提示，你都应该力求找到**完全概述预期行为的最小信息集**。（请注意，最小并不一定意味着简短；你仍然需要预先给智能体提供足够的信息，以确保它遵守期望的行为。）最好从使用可用的最佳模型测试最小提示开始，看看它在你的任务上的表现，然后根据初始测试期间发现的失败模式添加清晰的指令和示例来改进性能。

Regardless of how you decide to structure your system prompt, you should be striving for the minimal set of information that fully outlines your expected behavior. (Note that minimal does not necessarily mean short; you still need to give the agent sufficient information up front to ensure it adheres to the desired behavior.) It's best to start by testing a minimal prompt with the best model available to see how it performs on your task, and then add clear instructions and examples to improve performance based on failure modes found during initial testing.

工具允许智能体与其环境交互，并在工作时引入新的额外上下文。因为工具定义了智能体与其信息/行动空间之间的契约，所以工具促进效率极其重要，既要返回标记高效的信息，也要鼓励高效的智能体行为。

Tools allow agents to operate with their environment and pull in new, additional context as they work. Because tools define the contract between agents and their information/action space, it's extremely important that tools promote efficiency, both by returning information that is token efficient and by encouraging efficient agent behaviors.

在《使用 AI 智能体为 AI 智能体编写工具》一文中，我们讨论了构建被 LLM 充分理解且功能重叠最小的工具。类似于设计良好的代码库的功能，工具应该是自包含的、对错误稳健的，并且在其预期用途方面极其清晰。输入参数同样应该是描述性的、明确的，并发挥模型的固有优势。

In Writing tools for AI agents – with AI agents, we discussed building tools that are well understood by LLMs and have minimal overlap in functionality. Similar to the functions of a well-designed codebase, tools should be self-contained, robust to error, and extremely clear with respect to their intended use. Input parameters should similarly be descriptive, unambiguous, and play to the inherent strengths of the model.

我们看到的最常见失败模式之一是臃肿的工具集，涵盖了太多功能或导致关于使用哪个工具的模糊决策点。如果人类工程师无法明确说出在给定情况下应该使用哪个工具，就不能指望 AI 智能体做得更好。正如我们稍后将讨论的，为智能体策划最小可行的工具集也可以在长时间交互中更可靠地维护和修剪上下文。

One of the most common failure modes we see is bloated tool sets that cover too much functionality or lead to ambiguous decision points about which tool to use. If a human engineer can't definitively say which tool should be used in a given situation, an AI agent can't be expected to do better. As we'll discuss later, curating a minimal viable set of tools for the agent can also lead to more reliable maintenance and pruning of context over long interactions.

提供示例，也称为少样本提示，是一个众所周知的最佳实践，我们继续强烈建议采用。然而，团队经常会在提示中塞入一长串边缘情况，试图阐明 LLM 应该遵循的每一条可能的规则。我们不建议这样做。相反，我们建议努力策划一组多样化的典型示例，有效展示智能体的预期行为。对于 LLM 来说，示例是值一千字的"图片"。

Providing examples, otherwise known as **few-shot prompting**, is a well known best practice that we continue to strongly advise. However, teams will often stuff a laundry list of edge cases into a prompt in an attempt to articulate every possible rule the LLM should follow for a particular task. We do not recommend this. Instead, we recommend working to curate a set of diverse, canonical examples that effectively portray the expected behavior of the agent. For an LLM, examples are the "pictures" worth a thousand words.

我们对上下文不同组件（系统提示、工具、示例、消息历史等）的总体指导是要深思熟虑，保持上下文信息丰富但紧凑。现在让我们深入探讨在运行时动态检索上下文。

Our overall guidance across the different components of context (system prompts, tools, examples, message history, etc) is to be thoughtful and keep your context informative, yet tight. Now let's dive into dynamically retrieving context at runtime.

## 上下文检索和智能体搜索

Context retrieval and agentic search

在《构建有效的 AI 智能体》一文中，我们强调了基于 LLM 的工作流和智能体之间的区别。自从我们撰写该文章以来，我们倾向于对智能体采用一个简单的定义：**在循环中自主使用工具的 LLM**。

In Building effective AI agents, we highlighted the differences between LLM-based workflows and agents. Since we wrote that post, we've gravitated towards a simple definition for agents: LLMs autonomously using tools in a loop.

与客户合作时，我们看到该领域正在向这个简单范式趋同。随着底层模型变得更强大，智能体的自主程度可以扩展：更智能的模型允许智能体独立导航复杂问题空间并从错误中恢复。

Working alongside our customers, we've seen the field converging on this simple paradigm. As the underlying models become more capable, the level of autonomy of agents can scale: smarter models allow agents to independently navigate nuanced problem spaces and recover from errors.

我们现在看到工程师思考为智能体设计上下文的方式发生了转变。如今，许多 AI 原生应用采用某种形式的基于嵌入的推理前时间检索，以为智能体呈现重要上下文进行推理。随着该领域向更智能化的方法过渡，我们越来越多地看到团队用"即时"上下文策略来增强这些检索系统。

We're now seeing a shift in how engineers think about designing context for agents. Today, many AI-native applications employ some form of embedding-based pre-inference time retrieval to surface important context for the agent to reason over. As the field transitions to more agentic approaches, we increasingly see teams augmenting these retrieval systems with "just in time" context strategies.

智能体不是预先处理所有相关数据，而是采用"即时"方法**维护轻量级标识符（文件路径、存储的查询、网页链接等）**，并使用这些引用在运行时使用工具动态加载数据到上下文中。Anthropic 的智能体编码解决方案 Claude Code 使用这种方法对大型数据库执行复杂的数据分析。模型可以编写有针对性的查询、存储结果，并利用 Bash 命令如 head 和 tail 来分析大量数据，而无需将完整数据对象加载到上下文中。这种方法反映了人类认知：我们通常不会记忆整个信息语料库，而是引入外部组织和索引系统，如文件系统、收件箱和书签，以按需检索相关信息。

Rather than pre-processing all relevant data up front, agents built with the "just in time" approach maintain lightweight identifiers (file paths, stored queries, web links, etc.) and use these references to dynamically load data into context at runtime using tools. Anthropic's agentic coding solution Claude Code uses this approach to perform complex data analysis over large databases. The model can write targeted queries, store results, and leverage Bash commands like head and tail to analyze large volumes of data without ever loading the full data objects into context. This approach mirrors human cognition: we generally don't memorize entire corpuses of information, but rather introduce external organization and indexing systems like file systems, inboxes, and bookmarks to retrieve relevant information on demand.

除了存储效率之外，这些引用的**元数据**提供了一种有效优化行为的机制，无论是明确提供还是直观的。对于在文件系统中运行的智能体来说，tests 文件夹中名为 test_utils.py 的文件的存在，暗示的目的与位于 src/core_logic/ 中同名文件不同。文件夹层次结构、命名约定和时间戳都提供了重要信号，帮助人类和智能体理解如何以及何时利用信息。

Beyond storage efficiency, the metadata of these references provides a mechanism to efficiently refine behavior, whether explicitly provided or intuitive. To an agent operating in a file system, the presence of a file named test_utils.py in a tests folder implies a different purpose than a file with the same name located in src/core_logic/ Folder hierarchies, naming conventions, and timestamps all provide important signals that help both humans and agents understand how and when to utilize information.

让智能体自主导航和检索数据还能实现**渐进式披露**——换句话说，允许**智能体通过探索逐步发现相关上下文**。每次交互都会产生为下一个决策提供信息的上下文：文件大小暗示复杂性；命名约定暗示目的；时间戳可以作为相关性的代理。智能体可以逐层组装理解，在工作记忆中仅保留必要的内容，并利用记笔记策略实现额外的持久性。这种自我管理的上下文窗口使智能体专注于相关子集，而不是淹没在详尽但可能不相关的信息中。

Letting agents navigate and retrieve data autonomously also enables progressive disclosure—in other words, allows agents to incrementally discover relevant context through exploration. Each interaction yields context that informs the next decision: file sizes suggest complexity; naming conventions hint at purpose; timestamps can be a proxy for relevance. Agents can assemble understanding layer by layer, maintaining only what's necessary in working memory and leveraging note-taking strategies for additional persistence. This self-managed context window keeps the agent focused on relevant subsets rather than drowning in exhaustive but potentially irrelevant information.

当然，这是有权衡的：运行时探索比检索预计算数据慢。不仅如此，还需要有主见和深思熟虑的工程来确保 LLM 拥有正确的工具和启发式规则，以有效导航其信息环境。如果没有适当的指导，智能体可能会通过误用工具、追逐死胡同或未能识别关键信息来浪费上下文。

Of course, there's a trade-off: runtime exploration is slower than retrieving pre-computed data. Not only that, but opinionated and thoughtful engineering is required to ensure that an LLM has the right tools and heuristics for effectively navigating its information landscape. Without proper guidance, an agent can waste context by misusing tools, chasing dead-ends, or failing to identify key information.

在某些环境中，最有效的智能体可能采用混合策略，预先检索一些数据以提高速度，并自行决定进行进一步的自主探索。"正确"自主程度的决策边界取决于任务。Claude Code 是一个采用这种混合模型的智能体：CLAUDE.md 文件被直接放入上下文，而 glob 和 grep 等原语允许它导航其环境并即时检索文件，有效绕过了过时索引和复杂语法树的问题。

In certain settings, the most effective agents might employ a hybrid strategy, retrieving some data up front for speed, and pursuing further autonomous exploration at its discretion. The decision boundary for the 'right' level of autonomy depends on the task. Claude Code is an agent that employs this hybrid model: CLAUDE.md files are naively dropped into context up front, while primitives like glob and grep allow it to navigate its environment and retrieve files just-in-time, effectively bypassing the issues of stale indexing and complex syntax trees.

混合策略可能更适合内容变化较少的环境，例如法律或金融工作。随着模型能力的提高，智能体设计将趋向于让智能模型智能地行动，人工策划逐步减少。鉴于该领域的快速进展，"做最简单有效的事情"可能仍然是我们对在 Claude 之上构建智能体的团队的最佳建议。

The hybrid strategy might be better suited for contexts with less dynamic content, such as legal or finance work. As model capabilities improve, agentic design will trend towards letting intelligent models act intelligently, with progressively less human curation. Given the rapid pace of progress in the field, "do the simplest thing that works" will likely remain our best advice for teams building agents on top of Claude.

## 长期任务的上下文工程

Context engineering for long-horizon tasks

长期任务要求智能体在标记数超过 LLM 上下文窗口的行动序列中保持连贯性、上下文和目标导向行为。对于跨越数十分钟到数小时连续工作的任务，如大型代码库迁移或综合研究项目，智能体需要专门技术来克服上下文窗口大小限制。

Long-horizon tasks require agents to maintain coherence, context, and goal-directed behavior over sequences of actions where the token count exceeds the LLM's context window. For tasks that span tens of minutes to multiple hours of continuous work, like large codebase migrations or comprehensive research projects, agents require specialized techniques to work around the context window size limitation.

等待更大的上下文窗口可能看起来是一个明显的策略。但在可预见的未来，所有大小的上下文窗口都可能受到上下文污染和信息相关性问题的影响——至少在需要最强智能体性能的情况下是这样。为了使智能体能够在扩展的时间范围内有效工作，我们开发了一些直接解决这些上下文污染约束的技术：压缩、结构化记笔记和多智能体架构。

Waiting for larger context windows might seem like an obvious tactic. But it's likely that for the foreseeable future, context windows of all sizes will be subject to context pollution and information relevance concerns—at least for situations where the strongest agent performance is desired. To enable agents to work effectively across extended time horizons, we've developed a few techniques that address these context pollution constraints directly: compaction, structured note-taking, and multi-agent architectures.

**压缩**

Compaction

压缩是将接近上下文窗口限制的对话总结其内容，并用摘要重新启动新上下文窗口的实践。压缩通常作为上下文工程中推动更好长期连贯性的第一个杠杆。压缩的核心是以高保真方式提炼上下文窗口的内容，使智能体能够以最小的性能退化继续工作。

Compaction is the practice of taking a conversation nearing the context window limit, summarizing its contents, and reinitiating a new context window with the summary. Compaction typically serves as the first lever in context engineering to drive better long-term coherence. At its core, compaction distills the contents of a context window in a high-fidelity manner, enabling the agent to continue with minimal performance degradation.

例如，在 Claude Code 中，我们通过将消息历史传递给模型来实施此操作，以总结和压缩最关键的细节。模型保留**架构决策、未解决的错误和实现细节，同时丢弃冗余的工具输出或消息。然后，智能体可以使用这个压缩的上下文加上最近访问的五个文件继续工作。**用户获得连续性，无需担心上下文窗口限制。

In Claude Code, for example, we implement this by passing the message history to the model to summarize and compress the most critical details. The model preserves architectural decisions, unresolved bugs, and implementation details while discarding redundant tool outputs or messages. The agent can then continue with this compressed context plus the five most recently accessed files. Users get continuity without worrying about context window limitations.

压缩的艺术在于选择保留什么与丢弃什么，因为过于激进的压缩可能导致丢失微妙但关键的上下文，其重要性只在稍后才变得明显。对于实施压缩系统的工程师，我们建议在复杂的智能体轨迹上仔细调整提示。首先最大化召回率，以确保压缩提示捕获轨迹中的每一个相关信息，然后迭代以通过消除多余内容来提高精确度。

The art of compaction lies in the selection of what to keep versus what to discard, as overly aggressive compaction can result in the loss of subtle but critical context whose importance only becomes apparent later. For engineers implementing compaction systems, we recommend carefully tuning your prompt on complex agent traces. Start by maximizing recall to ensure your compaction prompt captures every relevant piece of information from the trace, then iterate to improve precision by eliminating superfluous content.

一个容易实现的多余内容示例是清除工具调用和结果——一旦工具在消息历史深处被调用，智能体为什么还需要再次看到原始结果？最安全、最轻触的压缩形式之一是工具结果清除，最近作为 Claude 开发者平台的一个功能推出。

An example of low-hanging superfluous content is clearing tool calls and results – once a tool has been called deep in the message history, why would the agent need to see the raw result again? One of the safest lightest touch forms of compaction is tool result clearing, most recently launched as a feature on the Claude Developer Platform.

**结构化记笔记**

Structured note-taking

结构化记笔记，或智能体记忆，是一种智能体定期将笔记写入上下文窗口之外的**持久化**内存的技术。这些笔记会在稍后时间被拉回到上下文窗口中。

Structured note-taking, or agentic memory, is a technique where the agent regularly writes notes persisted to memory outside of the context window. These notes get pulled back into the context window at later times.

这种策略以最小开销提供持久记忆。就像 Claude Code 创建待办事项列表，或你的自定义智能体维护 NOTES.md 文件一样，这种简单模式允许智能体跟踪复杂任务的进度，维护在数十次工具调用中否则会丢失的关键上下文和依赖关系。

This strategy provides persistent memory with minimal overhead. Like Claude Code creating a to-do list, or your custom agent maintaining a NOTES.md file, this simple pattern allows the agent to track progress across complex tasks, maintaining critical context and dependencies that would otherwise be lost across dozens of tool calls.

Claude 玩宝可梦展示了记忆如何在非编码领域转变智能体能力。智能体在数千个游戏步骤中保持精确计数——跟踪目标，例如"在过去的 1,234 步中，我一直在 1 号道路训练我的宝可梦，皮卡丘已经获得了 8 级，目标是 10 级。"在没有任何关于记忆结构的提示的情况下，它开发了已探索区域的地图，记住了解锁的关键成就，并维护了战斗策略的战略笔记，帮助它学习哪些攻击对不同对手最有效。

Claude playing Pokémon demonstrates how memory transforms agent capabilities in non-coding domains. The agent maintains precise tallies across thousands of game steps—tracking objectives like "for the last 1,234 steps I've been training my Pokémon in Route 1, Pikachu has gained 8 levels toward the target of 10." Without any prompting about memory structure, it develops maps of explored regions, remembers which key achievements it has unlocked, and maintains strategic notes of combat strategies that help it learn which attacks work best against different opponents.

在上下文重置后，智能体读取自己的笔记并继续多小时的训练序列或地牢探索。这种跨总结步骤的连贯性使得长期策略成为可能，如果仅将所有信息保留在 LLM 的上下文窗口中，这些策略将是不可能的。

After context resets, the agent reads its own notes and continues multi-hour training sequences or dungeon explorations. This coherence across summarization steps enables long-horizon strategies that would be impossible when keeping all the information in the LLM's context window alone.

作为我们 Sonnet 4.5 发布的一部分，我们在 Claude 开发者平台上发布了公开测试版的记忆工具，通过基于文件的系统更容易在上下文窗口之外存储和查询信息。这允许智能体随着时间推移建立知识库，跨会话维护项目状态，并引用以前的工作而无需将所有内容保留在上下文中。

As part of our Sonnet 4.5 launch, we released a memory tool in public beta on the Claude Developer Platform that makes it easier to store and consult information outside the context window through a file-based system. This allows agents to build up knowledge bases over time, maintain project state across sessions, and reference previous work without keeping everything in context.

**子智能体架构**

Sub-agent architectures

子智能体架构提供了另一种绕过上下文限制的方法。一个智能体不是试图在整个项目中维护状态，而是由专门的子智能体处理具有干净上下文窗口的聚焦任务。主智能体用高层计划进行协调，而子智能体执行深入的技术工作或使用工具查找相关信息。每个子智能体可能进行广泛探索，使用数万个甚至更多标记，但只返回其工作的浓缩、精炼摘要（通常为 1,000-2,000 个标记）。

Sub-agent architectures provide another way around context limitations. Rather than one agent attempting to maintain state across an entire project, specialized sub-agents can handle focused tasks with clean context windows. The main agent coordinates with a high-level plan while subagents perform deep technical work or use tools to find relevant information. Each subagent might explore extensively, using tens of thousands of tokens or more, but returns only a condensed, distilled summary of its work (often 1,000-2,000 tokens).

这种方法实现了清晰的关注点分离——详细的搜索上下文保持隔离在子智能体内，而主智能体专注于综合和分析结果。这种模式在《我们如何构建多智能体研究系统》中讨论过，在复杂研究任务上显示出比单智能体系统的显著改进。

This approach achieves a clear separation of concerns—the detailed search context remains isolated within sub-agents, while the lead agent focuses on synthesizing and analyzing the results. This pattern, discussed in How we built our multi-agent research system, showed a substantial improvement over single-agent systems on complex research tasks.

这些方法之间的选择取决于任务特征。例如：

The choice between these approaches depends on task characteristics. For example:

压缩为需要大量来回交互的任务维护对话流；
记笔记在具有明确里程碑的迭代开发中表现出色；
多智能体架构处理复杂的研究和分析，其中并行探索带来红利。

Compaction maintains conversational flow for tasks requiring extensive back-and-forth;
Note-taking excels for iterative development with clear milestones;
Multi-agent architectures handle complex research and analysis where parallel exploration pays dividends.

即使模型持续改进，在扩展交互中保持连贯性的挑战仍将是构建更有效智能体的核心。

Even as models continue to improve, the challenge of maintaining coherence across extended interactions will remain central to building more effective agents.

## 结论

Conclusion

上下文工程代表了我们使用 LLM 进行构建方式的根本转变。随着模型变得更强大，挑战不仅仅是制作完美的提示——而是在每一步深思熟虑地策划什么信息进入模型有限的注意力预算。无论你是为长期任务实施压缩、设计标记高效的工具，还是使智能体能够即时探索其环境，指导原则保持不变：找到最大化期望结果可能性的最小高信号标记集。

Context engineering represents a fundamental shift in how we build with LLMs. As models become more capable, the challenge isn't just crafting the perfect prompt—it's thoughtfully curating what information enters the model's limited attention budget at each step. Whether you're implementing compaction for long-horizon tasks, designing token-efficient tools, or enabling agents to explore their environment just-in-time, the guiding principle remains the same: find the smallest set of high-signal tokens that maximize the likelihood of your desired outcome.

我们概述的技术将随着模型的改进而继续发展。我们已经看到，更智能的模型需要更少的规定性工程，允许智能体以更大的自主性运行。但即使随着能力的扩展，将上下文视为宝贵的有限资源仍将是构建可靠、有效智能体的核心。

The techniques we've outlined will continue evolving as models improve. We're already seeing that smarter models require less prescriptive engineering, allowing agents to operate with more autonomy. But even as capabilities scale, treating context as a precious, finite resource will remain central to building reliable, effective agents.

今天就开始在 Claude 开发者平台进行上下文工程，并通过我们的记忆和上下文管理 cookbook 访问有用的提示和最佳实践。

Get started with context engineering in the Claude Developer Platform today, and access helpful tips and best practices via our memory and context management cookbook.

## 致谢

Acknowledgements

本文由 Anthropic 应用 AI 团队撰写：Prithvi Rajasekaran、Ethan Dixon、Carly Ryan 和 Jeremy Hadfield，团队成员 Rafi Ayub、Hannah Moran、Cal Rueb 和 Connor Jennings 做出了贡献。特别感谢 Molly Vorwerck、Stuart Ritchie 和 Maggie Vo 的支持。

Written by Anthropic's Applied AI team: Prithvi Rajasekaran, Ethan Dixon, Carly Ryan, and Jeremy Hadfield, with contributions from team members Rafi Ayub, Hannah Moran, Cal Rueb, and Connor Jennings. Special thanks to Molly Vorwerck, Stuart Ritchie, and Maggie Vo for their support.
