# AI 智能体的上下文工程：构建 Manus 的经验教训

Context Engineering for AI Agents: Lessons from Building Manus---
https://manus.im/blog/Context-Engineering-for-AI-Agents-Lessons-from-Building-Manus

2025年7月18日

2025/7/18

作者：Yichao 'Peak' Ji

在项目最开始时，我和团队面临一个关键决策：我们应该使用开源基础模型训练一个端到端的智能体模型，还是在前沿模型的现有能力之上构建智能体？

At the very beginning of the project, my team and I faced a key decision: should we train an end-to-end agentic model using open-source foundations, or build an agent on top of the abilities of frontier models?

回顾我在 NLP 领域的第一个十年，我们没有这种选择的奢侈。在遥远的年代（是的，已经过去七年了），模型必须经过微调和评估才能迁移到新任务。这个过程每次迭代通常需要数周时间，尽管那些模型与今天的 LLM 相比非常小。对于快速发展的应用，尤其是在产品市场契合度（PMF）之前，这种缓慢的反馈循环是致命的。这是我上一个创业项目的痛苦教训，当时我为文本分类和语义搜索从头训练模型。然后出现了 GPT-3 和 BERT，我的自研模型一夜之间变得无关紧要。讽刺的是，正是这些模型标志着上下文学习的开始，以及一条全新的前进道路。

Back in my first decade in NLP, we didn't have the luxury of that choice. In the distant days of (yes, it's been seven years), models had to be fine-tuned—and evaluated—before they could transfer to a new task. That process often took weeks per iteration, even though the models were tiny compared to today's LLMs. For fast-moving applications, especially pre–PMF, such slow feedback loops are a deal-breaker. That was a bitter lesson from my last startup, where I trained models from scratch for text classification and semantic search. Then came GPT-3 and BERT, and my in-house models became irrelevant overnight. Ironically, those same models marked the beginning of in-context learning—and a whole new path forward.

这个来之不易的教训让选择变得清晰：Manus 将押注于上下文工程。这使我们能够在数小时而不是数周内发布改进，并使我们的产品与底层模型保持正交关系：如果模型进步是涨潮，我们希望 Manus 是船，而不是固定在海床上的柱子。

That hard-earned lesson made the choice clear: Manus would bet on context engineering. This allows us to ship improvements in hours instead of weeks, and kept our product orthogonal to the underlying models: If model progress is the rising tide, we want Manus to be the boat, not the pillar stuck to the seabed.

然而，事实证明，上下文工程绝非简单直接。这是一门实验科学——我们已经重建了四次智能体框架，每次都是在发现更好的上下文塑造方法之后。我们亲切地将这种架构搜索、提示调整和经验猜测的手动过程称为"随机梯度下降"（Stochastic Graduate Descent）。这不优雅，但有效。

Still, context engineering turned out to be anything but straightforward. It's an experimental science—and we've rebuilt our agent framework four times, each time after discovering a better way to shape context. We affectionately refer to this manual process of architecture searching, prompt fiddling, and empirical guesswork as "Stochastic Graduate Descent". It's not elegant, but it works.

这篇文章分享了我们通过自己的"SGD"达到的局部最优解。如果你正在构建自己的 AI 智能体，我希望这些原则能帮助你更快地收敛。

This post shares the local optima we arrived at through our own "SGD". If you're building your own AI agent, I hope these principles help you converge faster.

## 围绕 KV 缓存进行设计

Design Around the KV-Cache

如果我必须选择一个指标，我会说 **KV 缓存命中率是生产阶段 AI 智能体最重要的单一指标**。它直接影响延迟和成本。要理解原因，让我们看看智能体的工作方式：

If I had to choose just one metric, I'd argue that the KV-cache hit rate is the single most important metric for a production-stage AI agent. It directly affects both latency and cost. To understand why, let's look at how agents operate:

在收到用户输入后，智能体通过一系列工具使用来完成任务。在每次迭代中，模型根据当前上下文从预定义的操作空间中选择一个动作。然后在环境中执行该动作（例如，Manus 的虚拟机沙箱）以产生观察结果。动作和观察结果被附加到上下文中，形成下一次迭代的输入。这个循环持续进行，直到任务完成。

After receiving a user input, the agent proceeds through a chain of tool uses to complete the task. In each iteration, the model selects an action from a predefined action space based on the current context. That action is then executed in the environment (e.g., Manus's virtual machine sandbox) to produce an observation. The action and observation are appended to the context, forming the input for the next iteration. This loop continues until the task is complete.

可以想象，上下文随着每一步而增长，而输出——通常是结构化的函数调用——保持相对较短。这使得智能体中预填充（prefilling）与解码（decoding）之间的比率与聊天机器人相比高度倾斜。例如，在 Manus 中，平均输入与输出 token 比率约为 100:1。

As you can imagine, the context grows with every step, while the output—usually a structured function call—remains relatively short. This makes the ratio between prefilling and decoding highly skewed in agents compared to chatbots. In Manus, for example, the average input-to-output token ratio is around 100:1.

幸运的是，具有相同前缀的上下文可以利用 KV 缓存，这大大减少了首 token 时间（TTFT）和推理成本——无论你是使用自托管模型还是调用推理 API。而且我们说的不是小额节省：例如，使用 Claude Sonnet，缓存的输入 token 成本为 0.30 美元/百万 token，而未缓存的为 3 美元/百万 token——相差 10 倍。

Fortunately, contexts with identical prefixes can take advantage of KV-cache, which drastically reduces time-to-first-token (TTFT) and inference cost—whether you're using a self-hosted model or calling an inference API. And we're not talking about small savings: with Claude Sonnet, for instance, cached input tokens cost 0.30 USD/MTok, while uncached ones cost 3 USD/MTok—a 10x difference.

从上下文工程的角度来看，提高 KV 缓存命中率涉及几个关键实践：

From a context engineering perspective, improving KV-cache hit rate involves a few key practices:

**保持提示前缀稳定。** 由于 LLM 的自回归特性，即使是单个 token 的差异也可能从该 token 开始使缓存失效。一个常见错误是在系统提示开头包含时间戳——尤其是精确到秒的时间戳。当然，这让模型能告诉你当前时间，但也会破坏你的缓存命中率。

Keep your prompt prefix stable. Due to the autoregressive nature of LLMs, even a single-token difference can invalidate the cache from that token onward. A common mistake is including a timestamp—especially one precise to the second—at the beginning of the system prompt. Sure, it lets the model tell you the current time, but it also kills your cache hit rate.

**使上下文仅追加。** 避免修改之前的动作或观察结果。确保序列化是确定性的。许多编程语言和库在序列化 JSON 对象时不保证稳定的键顺序，这可能会悄悄破坏缓存。

Make your context append-only. Avoid modifying previous actions or observations. Ensure your serialization is deterministic. Many programming languages and libraries don't guarantee stable key ordering when serializing JSON objects, which can silently break the cache.

**需要时显式标记缓存断点。** 一些模型提供商或推理框架不支持自动增量前缀缓存，而是需要在上下文中手动插入缓存断点。分配这些断点时，要考虑潜在的缓存过期，至少确保断点包括系统提示的结尾。

Mark cache breakpoints explicitly when needed. Some model providers or inference frameworks don't support automatic incremental prefix caching, and instead require manual insertion of cache breakpoints in the context. When assigning these, account for potential cache expiration and at minimum, ensure the breakpoint includes the end of the system prompt.

此外，如果你使用像 vLLM 这样的框架自托管模型，请确保启用了前缀缓存，并且使用会话 ID 等技术在分布式工作节点之间一致地路由请求。

Additionally, if you're self-hosting models using frameworks like vLLM, make sure prefix caching is enabled, and that you're using techniques like session IDs to route requests consistently across distributed workers.

## 遮蔽，而不是移除

Mask, Don't Remove

随着智能体承担更多能力，其操作空间自然变得更加复杂——简单来说，工具数量激增。最近 MCP（模型上下文协议）的流行只会火上浇油。如果你允许用户可配置的工具，相信我：总会有人将数百个神秘工具插入你精心策划的操作空间。结果是，模型更有可能选择错误的动作或采取低效的路径。简而言之，你武装到牙齿的智能体变笨了。

As your agent takes on more capabilities, its action space naturally grows more complex—in plain terms, the number of tools explodes. The recent popularity of MCP only adds fuel to the fire. If you allow user-configurable tools, trust me: someone will inevitably plug hundreds of mysterious tools into your carefully curated action space. As a result, the model is more likely to select the wrong action or take an inefficient path. In short, your heavily armed agent gets dumber.

一个自然的反应是设计动态操作空间——也许使用类似 RAG 的东西按需加载工具。我们在 Manus 中也尝试过。但我们的实验表明了一个明确的规则：**除非绝对必要，避免在迭代过程中动态添加或删除工具。** 主要有两个原因：

A natural reaction is to design a dynamic action space—perhaps loading tools on demand using something RAG-like. We tried that in Manus too. But our experiments suggest a clear rule: unless absolutely necessary, avoid dynamically adding or removing tools mid-iteration. There are two main reasons for this:

在大多数 LLM 中，工具定义在序列化后位于上下文前部，通常在系统提示之前或之后。因此，任何更改都会使所有后续动作和观察结果的 KV 缓存失效。

In most LLMs, tool definitions live near the front of the context after serialization, typically before or after the system prompt. So any change will invalidate the KV-cache for all subsequent actions and observations.

当之前的动作和观察结果仍然引用当前上下文中不再定义的工具时，模型会感到困惑。如果没有工具使用链（tool use chains），这通常会导致模式违规或幻觉动作。

When previous actions and observations still refer to tools that are no longer defined in the current context, the model gets confused. Without tool use chains, this often leads to schema violations or hallucinated actions.

为了在改进动作选择的同时解决这个问题，Manus 使用上下文感知的 logistate machine 来管理工具可用性。它不是移除工具，而是在解码期间遮蔽 token logits，以根据当前上下文防止（或强制）选择某些动作。

To solve this while still improving action selection, Manus uses a context-aware state machine to manage tool availability. Rather than removing tools, it masks the token logits during decoding to prevent (or enforce) the selection of certain actions based on the current context.

实际上，大多数模型提供商和推理框架都支持某种形式的响应预填充，允许你在不修改工具定义的情况下约束操作空间。通常有三种函数调用模式（我们将使用 NousResearch 的 Hermes 格式作为例子）：

In practice, most model providers and inference frameworks support some form of response prefill, which allows you to constrain the action space without modifying the tool definitions. There are generally three modes of function calling (we'll use the Hermes format from NousResearch as an example):

**Auto** – 模型可以选择是否调用函数。通过仅预填充回复前缀实现：`<|im_start|>assistant`

Auto – The model may choose to call a function or not. Implemented by prefilling only the reply prefix: `<|im_start|>assistant`

**Required** – 模型必须调用函数，但选择不受约束。通过预填充到工具调用 token 实现：`<|im_start|>assistant<tool_call>`

Required – The model must call a function, but the choice is unconstrained. Implemented by prefilling up to tool call token: `<|im_start|>assistant<tool_call>`

**Specified** – 模型必须从特定子集调用函数。通过预填充到函数名称开头实现：`<|im_start|>assistant<tool_call>{"name": "browser_`

Specified – The model must call a function from a specific subset. Implemented by prefilling up to the beginning of the function name: `<|im_start|>assistant<tool_call>{"name": "browser_`

使用这种方法，我们通过直接遮蔽 token logits 来约束动作选择。例如，当用户提供新输入时，Manus 必须立即回复而不是采取动作。我们还特意设计了具有一致前缀的动作名称——例如，所有与浏览器相关的工具都以 `browser_` 开头，命令行工具以 `shell_` 开头。这使我们能够轻松强制智能体在给定状态下只从某组工具中选择，而无需使用有状态的 logits 处理器。

Using this, we constrain action selection by masking token logits directly. For example, when the user provides a new input, Manus must reply immediately instead of taking an action. We've also deliberately designed action names with consistent prefixes—e.g., all browser-related tools start with `browser_`, and command-line tools with `shell_`. This allows us to easily enforce that the agent only chooses from a certain group of tools at a given state without using stateful logits processors.

这些设计有助于确保 Manus 智能体循环保持稳定——即使在模型驱动的架构下也是如此。

These designs help ensure that the Manus agent loop remains stable—even under a model-driven architecture.

## 使用文件系统作为上下文

Use the File System as Context

现代前沿 LLM 现在提供 128K token 或更多的上下文窗口。但在实际的智能体场景中，这通常是不够的，有时甚至是一种负担。有三个常见的痛点：

Modern frontier LLMs now offer context windows of 128K tokens or more. But in real-world agentic scenarios, that's often not enough, and sometimes even a liability. There are three common pain points:

观察结果可能非常庞大，尤其是当智能体与网页或 PDF 等非结构化数据交互时。很容易超出上下文限制。

Observations can be huge, especially when agents interact with unstructured data like web pages or PDFs. It's easy to blow past the context limit.

即使窗口在技术上支持，模型性能也往往会在超过某个上下文长度后下降。

Model performance tends to degrade beyond a certain context length, even if the window technically supports it.

长输入成本高昂，即使使用前缀缓存。你仍然需要为传输和预填充每个 token 付费。

Long inputs are expensive, even with prefix caching. You're still paying to transmit and prefill every token.

为了应对这一点，许多智能体系统实施上下文截断或压缩策略。但过于激进的压缩不可避免地导致信息丢失。问题是根本性的：智能体从本质上必须根据所有先前状态预测下一个动作——你无法可靠地预测哪个观察结果可能在十步之后变得关键。从逻辑角度来看，任何不可逆的压缩都存在风险。

To deal with this, many agent systems implement context truncation or compression strategies. But overly aggressive compression inevitably leads to information loss. The problem is fundamental: an agent, by nature, must predict the next action based on all prior state—and you can't reliably predict which observation might become critical ten steps later. From a logical standpoint, any irreversible compression carries risk.

这就是为什么我们将文件系统视为 Manus 中的终极上下文：**大小无限、本质上持久化，并且可由智能体本身直接操作**。模型学会按需写入和读取文件——使用文件系统不仅作为存储，而且作为结构化的外部化内存。

That's why we treat the file system as the ultimate context in Manus: unlimited in size, persistent by nature, and directly operable by the agent itself. The model learns to write to and read from files on demand—using the file system not just as storage, but as structured, externalized memory.

我们的压缩策略总是设计为可恢复的。例如，只要保留 URL，网页内容就可以从上下文中删除；只要沙箱中仍然有文档路径，文档内容就可以省略。这允许 Manus 在不永久丢失信息的情况下缩短上下文长度。

Our compression strategies are always designed to be restorable. For instance, the content of a web page can be dropped from the context as long as the URL is preserved, and a document's contents can be omitted if its path remains available in the sandbox. This allows Manus to shrink context length without permanently losing information.

在开发这个功能时，我发现自己在想象状态空间模型（SSM）在智能体设置中有效工作需要什么。与 Transformer 不同，SSM 缺乏完整的注意力机制，并且在长程反向依赖方面存在困难。但如果它们能够掌握基于文件的内存——将长期状态外部化而不是保存在上下文中——那么它们的速度和效率可能会解锁新一类智能体。智能体 SSM 可能是 Neural Turing Machines 的真正继承者。

While developing this feature, I found myself imagining what it would take for a State Space Model (SSM) to work effectively in an agentic setting. Unlike Transformers, SSMs lack full attention and struggle with long-range backward dependencies. But if they could master file-based memory—externalizing long-term state instead of holding it in context—then their speed and efficiency might unlock a new class of agents. Agentic SSMs could be the real successors to Neural Turing Machines.

## 通过复述操纵注意力

Manipulate Attention Through Recitation

如果你使用过 Manus，你可能注意到一些有趣的事情：在处理复杂任务时，它倾向于创建一个 `todo.md` 文件——并在任务进行时逐步更新它，勾选已完成的项目。

If you've worked with Manus, you've probably noticed something curious: when handling complex tasks, it tends to create a `todo.md` file—and update it step-by-step as the task progresses, checking off completed items.

这不仅仅是可爱的行为——这是一种操纵注意力的深思熟虑的机制。

That's not just cute behavior—it's a deliberate mechanism to manipulate attention.

Manus 中的典型任务平均需要大约 50 次工具调用。这是一个很长的循环——由于 Manus 依赖 LLM 做决策，它容易偏离主题或忘记早期目标，尤其是在长上下文或复杂任务中。

A typical task in Manus requires around 50 tool calls on average. That's a long loop—and since Manus relies on LLMs for decision-making, it's vulnerable to drifting off-topic or forgetting earlier goals, especially in long contexts or complicated tasks.

通过不断重写待办事项列表，**Manus 正在将其目标复述到上下文末尾**。这将全局计划推入模型的近期注意力范围，避免"中间迷失"问题并减少目标不一致。实际上，它使用自然语言将自己的注意力偏向任务目标——无需特殊的架构更改。

By constantly rewriting the todo list, Manus is reciting its objectives into the end of the context. This pushes the global plan into the model's recent attention span, avoiding "lost-in-the-middle" issues and reducing goal misalignment. In effect, it's using natural language to bias its own focus toward the task objective—without needing special architectural changes.

## 保留错误的内容

Keep the Wrong Stuff In

智能体会犯错。这不是 bug——这是现实。语言模型产生幻觉，环境返回错误，外部工具行为不当，意外的边缘情况总是出现。在多步骤任务中，失败不是例外；它是循环的一部分。

Agents make mistakes. That's not a bug—it's reality. Language models hallucinate, environments return errors, external tools misbehave, and unexpected edge cases show up all the time. In multi-step tasks, failure is not the exception; it's part of the loop.

然而，一个常见的冲动是隐藏这些错误：清理轨迹、重试动作，或重置模型状态并将其留给神奇的"自我修复"。这感觉更安全、更可控。但这是有代价的：**抹去失败会移除证据。没有证据，模型就无法适应。**

And yet, a common impulse is to hide these errors: clean up the trace, retry the action, or reset the model's state and leave it to the magical "self-healing". That feels safer, more controlled. But it comes at a cost: Erasing failure removes evidence. And without evidence, the model can't adapt.

根据我们的经验，改进智能体行为最有效的方法之一看似简单：**在上下文中保留错误的转折**。当模型看到失败的动作——以及由此产生的观察或堆栈跟踪——它会隐式更新其内部信念。这使其先验远离类似的动作，减少重复相同错误的机会。

In our experience, one of the most effective ways to improve agent behavior is deceptively simple: leave the wrong turns in the context. When the model sees a failed action—and the resulting observation or stack trace—it implicitly updates its internal beliefs. This shifts its prior away from similar actions, reducing the chance of repeating the same mistake.

实际上，我们认为**错误恢复是真正智能体行为的最清晰指标之一**。然而，它在大多数学术工作和公共基准测试中仍然代表性不足，这些基准测试通常关注理想条件下的任务成功。

In fact, we believe error recovery is one of the clearest indicators of true agentic behavior. Yet it's still underrepresented in most academic work and public benchmarks, which often focus on task success under ideal conditions.

## 不要被少样本困住

Don't Get Few-Shotted

少样本学习（Few-shot prompting）是改进 LLM 输出的常用技术。但在智能体系统中，它可能会以微妙的方式适得其反。

Few-shot prompting is a common technique for improving LLM outputs. But in agent systems, it can backfire in subtle ways.

语言模型是出色的模仿者；它们模仿上下文中的行为模式。如果你的上下文充满了类似的过去动作-观察对，模型往往会遵循该模式，即使它不再是最优的。

Language models are excellent mimics; they imitate the pattern of behavior in the context. If your context is full of similar past action-observation pairs, the model will tend to follow that pattern, even when it's no longer optimal.

这在涉及重复决策或动作的任务中可能很危险。例如，当使用 Manus 帮助审查 20 份简历时，智能体经常陷入节奏——仅仅因为这是它在上下文中看到的，就重复类似的动作。这导致漂移、过度泛化，有时还有幻觉。

This can be dangerous in tasks that involve repetitive decisions or actions. For example, when using Manus to help review a batch of 20 resumes, the agent often falls into a rhythm—repeating similar actions simply because that's what it sees in the context. This leads to drift, overgeneralization, or sometimes hallucination.

解决方法是增加多样性。Manus 在动作和观察结果中引入少量结构化变化——不同的序列化模板、替代措辞、顺序或格式的轻微噪声。这种受控的随机性有助于打破模式并调整模型的注意力。

The fix is to increase diversity. Manus introduces small amounts of structured variation in actions and observations—different serialization templates, alternate phrasing, minor noise in order or formatting. This controlled randomness helps break the pattern and tweaks the model's attention.

换句话说，不要让少样本学习把自己困在车辙里。你的上下文越统一，你的智能体就越脆弱。

In other words, don't few-shot yourself into a rut. The more uniform your context, the more brittle your agent becomes.

## 结论

Conclusion

上下文工程仍然是一门新兴科学——但对于智能体系统来说，它已经至关重要。模型可能变得更强、更快、更便宜，但再多的原始能力也无法取代对内存、环境和反馈的需求。你如何塑造上下文最终定义了你的智能体如何行为：它运行多快、恢复得多好，以及扩展多远。

Context engineering is still an emerging science—but for agent systems, it's already essential. Models may be getting stronger, faster, and cheaper, but no amount of raw capability replaces the need for memory, environment, and feedback. How you shape the context ultimately defines how your agent behaves: how fast it runs, how well it recovers, and how far it scales.

在 Manus，我们通过反复重写、死胡同和数百万用户的实际测试学到了这些教训。我们在这里分享的内容都不是普遍真理——但这些是对我们有效的模式。如果它们能帮助你避免哪怕一次痛苦的迭代，那么这篇文章就完成了它的工作。

At Manus, we've learned these lessons through repeated rewrites, dead ends, and real-world testing across millions of users. None of what we've shared here is universal truth—but these are the patterns that worked for us. If they help you avoid even one painful iteration, then this post did its job.

智能体的未来将一次构建一个上下文。好好设计它们。

The agentic future will be built one context at a time. Engineer them well.
