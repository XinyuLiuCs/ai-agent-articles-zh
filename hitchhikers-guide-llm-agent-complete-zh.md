# LLM Agent 实用指南

*The Hitchhikers Guide to LLM Agent*---
https://saurabhalone.com/blog/agent

Manfred Mohr - P-197 Cubic Limit II (1979)

Figure 1. Manfred Mohr — P-197 Cubic Limit II (1979). Algorithmic art exploring systematic variations of the cube.

图 1. Manfred Mohr — P-197 Cubic Limit II (1979)。探索立方体系统性变化的算法艺术。

---

1. Hakken is an open-source CLI coding agent I built from scratch. Check it out on GitHub.

1. Hakken 是我从零开始构建的开源 CLI 编码代理。可以在 GitHub 上查看。

I spent the last few months building Hakken1, a coding agent from scratch. So in this blog I'm going to share what I learned; what actually matters when building agents that work great.

过去几个月我从零开始构建了 Hakken1，一个编码代理。在这篇博客中，我将分享我学到的东西；在构建真正好用的代理时，什么才是真正重要的。

You might think there are lots of other coding agents which are absolutely the best, like Claude Code, Codex, and OpenCode. So why do we need another one? Tbh I just have this curious mind that just want to understand everything from scratch. So I built this agent out of curiosity. Some sections might be opinionated, but that's how I want to share it. Everything!

你可能会想，已经有很多其他绝对最好的编码代理了，比如 Claude Code、Codex 和 OpenCode。那么为什么还需要另一个呢？说实话，我只是有一颗好奇的心，想从头开始理解一切。所以我出于好奇心构建了这个代理。有些部分可能会比较主观，但这就是我想分享的方式。全部！

So, let's get started! Grab your Diet Coke and wipe away your tears if you're a Ferrari fan!

那么，让我们开始吧！拿起你的健怡可乐，如果你是法拉利车迷的话，擦干你的眼泪！

This blog is divided into following five important sections:

本博客分为以下五个重要部分：

- What is an LLM and Inference - Before getting into LLM Agent lets take a quick look into some fundamentals of LLMs.
- Context Engineering is Everything - The Most Important for building reliable agentic system.
- Evaluation: Build It First - We will explore how we can evaluate our agent.
- What About Memory? - We will explore why agents need memory, how to implement them and why it's overhyped.
- The Subagent Pattern - We will see where using multiple agent is useful.

- 什么是 LLM 和推理 - 在深入 LLM Agent 之前，让我们快速了解一些 LLM 的基础知识。
- 上下文工程就是一切 - 这是构建可靠代理系统最重要的部分。
- 评估：先构建它 - 我们将探讨如何评估我们的代理。
- 记忆呢？- 我们将探讨为什么代理需要记忆，如何实现它们，以及为什么它被过度炒作了。
- 子代理模式 - 我们将看到使用多个代理在哪里有用。

## What is an LLM and Inference

## 什么是 LLM 和推理

Let's take a look into some fundamental concepts which I think these concepts are very important to understand if you are working with LLMs. I am just giving an overview but you should study more deeply about it.

让我们来看看一些基本概念，我认为如果你在使用 LLM，理解这些概念非常重要。我只是给出一个概述，但你应该更深入地研究它。

### What Is An LLM Agent?

### 什么是 LLM Agent？

In short, LLM agent is an LLM in a feedback loop with tools to interact with its environment. You can think of it like this; you give it a task in natural language and it breaks it down and then it calls tools to perform actions and then observes the result and keeps going until the task is done or model fucks up.

简而言之，LLM agent 是一个处于反馈循环中的 LLM，配备工具来与其环境交互。你可以这样想；你用自然语言给它一个任务，它将其分解，然后调用工具执行操作，然后观察结果并持续进行，直到任务完成或模型搞砸。

The core loop looks something like this:

核心循环看起来像这样：

Agent core loop diagram

代理核心循环图

Figure 2. The core agent loop — observe, think, act, repeat until task completion or failure.

图 2. 核心代理循环 — 观察、思考、行动，重复直到任务完成或失败。

Agents should work autonomously; handling any length tasks without human interruption right? That's what agent means right?

代理应该自主工作；处理任何长度的任务而无需人工干预，对吧？这就是代理的含义，对吧？

But there is one problem: when given long-horizon tasks, it starts creating a mess at some point and then there is no going back from that. This is the biggest unsolved problem in agent land right now. Models are getting better, but they still can't handle 100+ step workflows reliably.

但有一个问题：当给定长时程任务时，它会在某个时刻开始制造混乱，然后就无法挽回了。这是目前代理领域最大的未解决问题。模型正在变得更好，但它们仍然无法可靠地处理 100+ 步的工作流程。

Does this mean they're completely useless for long-horizon tasks? No, they're not completely useless. With proper context management; you can use these models to build agents that can be useful for long-horizon tasks. I'll explain what context engineering is and why it matters. First let's see the basic Agent loop that makes all of this work.

这是否意味着它们对于长时程任务完全无用？不，它们并非完全无用。通过适当的上下文管理；你可以使用这些模型来构建对长时程任务有用的代理。我将解释什么是上下文工程以及为什么它很重要。首先让我们看看使所有这些工作的基本代理循环。

Here's the simplest agent loop you can build:

这是你可以构建的最简单的代理循环：

```python
async def agent_loop(task: str):
    messages = [
        {"role": "system", "content": SYSTEM_PROMPT},
        {"role": "user", "content": task}
    ]

    while True:
        response = await llm.chat(messages)
        messages.append({"role": "assistant", "content": response})

        # Check if model wants to use tools
        if response.tool_calls:
            for tool_call in response.tool_calls:
                result = await execute_tool(tool_call)
                messages.append({
                    "role": "tool",
                    "tool_call_id": tool_call.id,
                    "content": result
                })
        else:
            # No tool calls = task complete
            return response.content
```

That's it. Call LLM → check for tools → execute tools → repeat2. Every agent framework is just this loop with extra steps.

就是这样。调用 LLM → 检查工具 → 执行工具 → 重复2。每个代理框架都只是这个循环加上额外的步骤。

2. Real world is messier. You need error handling, retries, timeouts, rate limits, token counting, streaming... but the core idea is exactly this simple. Don't let frameworks convince you otherwise.

2. 现实世界更混乱。你需要错误处理、重试、超时、速率限制、令牌计数、流式传输……但核心思想正是这么简单。不要让框架让你相信其他的。

### Why We Need LLM Agent?

### 为什么我们需要 LLM Agent？

Simple cause LLM is just a next word predictor; it doesn't know what to do next. It needs to be told what to do next. And that's where the agent comes in.

简单原因是 LLM 只是一个下一个词预测器；它不知道接下来要做什么。它需要被告知接下来要做什么。这就是代理的作用所在。

You can see in the following diagram how the LLM works.

你可以在下图中看到 LLM 是如何工作的。

LLM Inference Architecture

LLM 推理架构

Figure 3. The complete LLM inference pipeline — from API call to token generation, showing prefill and decode phases.

图 3. 完整的 LLM 推理管道 — 从 API 调用到令牌生成，展示预填充和解码阶段。

Here's what actually happens when you call an LLM API:

当你调用 LLM API 时实际发生的事情：

**Prefill Phase**

**预填充阶段**

You give prompt or input "what is the color of sky" to llm, first it get converted into numbers [12840, 374, 279, 1933, 315, 13180, 374] and then it get converted into embeddings and then processed through all transformer layers in parallel. After that model computes attention for all input tokens at once and outputs logits for the next token position.

你给 llm 提示或输入"天空的颜色是什么"，首先它被转换为数字 [12840, 374, 279, 1933, 315, 13180, 374]，然后被转换为嵌入，然后并行处理所有 transformer 层。之后，模型一次性计算所有输入令牌的注意力，并输出下一个令牌位置的 logits。

All your input tokens get processed at the same time. If you have 1000 tokens in your prompt, all 1000 go through the model together. It's like batch processing.

你的所有输入令牌同时被处理。如果你的提示中有 1000 个令牌，所有 1000 个令牌会一起通过模型。这就像批处理一样。

So whether you send 100 or 1000 tokens, the GPU can process them in parallel using all its cores. We call this compute-bound because we're actually using the GPU's compute power efficiently.

所以无论你发送 100 还是 1000 个令牌，GPU 都可以使用其所有核心并行处理它们。我们称之为计算受限，因为我们实际上正在有效地使用 GPU 的计算能力。

**Decode Phase**

**解码阶段**

3. Autoregressive = each new token depends on ALL previous tokens. Like writing a sentence where each word you write changes what word comes next. You can't parallelize this shit.

3. 自回归 = 每个新令牌都依赖于所有先前的令牌。就像写句子一样，你写的每个词都会改变接下来的词。你无法并行化这玩意儿。

After Prefill phase the model start generating tokens one by one autoregressively this known as Decode Phase3. And each new tokens depends on all previous tokens. Here we use KV-cache4 to avoid recomputing everything. We store key/value matrices from previous tokens so you only need to process the new token. Without this, attention would be O(n²) for every single token. With it, it's O(n).

在预填充阶段之后，模型开始自回归地逐个生成令牌，这被称为解码阶段3。每个新令牌都依赖于所有先前的令牌。在这里我们使用 KV-cache4 来避免重新计算所有内容。我们存储来自先前令牌的键/值矩阵，因此你只需要处理新令牌。没有这个，注意力对于每个单独的令牌将是 O(n²)。有了它，就是 O(n)。

4. KV = Key-Value. In transformer attention, you compute Query, Key, Value matrices. The KV-cache stores the Key and Value matrices so you don't recompute them for every fucking token. Saves insane amount of compute.

4. KV = 键-值。在 transformer 注意力中，你计算查询、键、值矩阵。KV-cache 存储键和值矩阵，所以你不必为每个该死的令牌重新计算它们。节省了大量的计算。

This phase is memory-bound - because each new token requires streaming the entire KV cache from memory with very little computation per byte and cause of autoregressive nature we cannot do parallelism.

这个阶段是记忆受限的 - 因为每个新令牌都需要从记忆中流式传输整个 KV 缓存，每个字节的计算量很少，而且由于自回归特性，我们无法进行并行化。

Then **sampling**: softmax converts logits to probabilities and you pick the next token (greedy, top-k, temperature, whatever). Repeat until you hit max length or the <EOS> token.

然后采样：softmax 将 logits 转换为概率，你选择下一个令牌（贪婪、top-k、温度，随便什么）。重复直到达到最大长度或 <EOS> 令牌。

Okay lets get back to the Agent.

好的，让我们回到代理。

In the age of LLMs and building AI Agents, it feels like we're still playing with raw HTML & CSS and figuring out how to fit these together to make a good experience. No single approach to building agents has become the standard yet, besides some of the absolute basics — Cognition.

在 LLM 和构建 AI Agent 的时代，感觉我们仍在玩原始的 HTML & CSS，并试图弄清楚如何将它们组合在一起以创造良好的体验。除了一些绝对基础的东西 — Cognition，还没有任何一种构建代理的方法成为标准。

AND I think everyone who's building AI agents should go through the 12-factor-agents repo by Dex5. He has done a great job explaining the architecture and principles to build reliable agents.

我认为每个构建 AI 代理的人都应该阅读 Dex5 的 12-factor-agents 仓库。他在解释构建可靠代理的架构和原则方面做得很好。

5. Dex is from HumanLayer. The 12-factor thing is inspired by the famous "12 Factor App" methodology for building SaaS. Same vibes but for agents. Go read it, seriously.

5. Dex 来自 HumanLayer。12-factor 的概念受到著名的构建 SaaS 的"12 Factor App"方法论的启发。同样的氛围，但用于代理。认真去读一读。

Okay as we said that the most important thing to building reliable agent is context engineering so let's see how to do it.

好的，正如我们所说，构建可靠代理最重要的事情是上下文工程，所以让我们看看如何做到这一点。

## Context Engineering is Everything

## 上下文工程就是一切

Okay lets start with - What is context window? It's basically the input that you give to the LLM. That means system prompt, user prompt, tool description, history, memory, tool output, everything that you give LLM to generate good output.

好的，让我们从什么是上下文窗口开始？它基本上是你给 LLM 的输入。这意味着系统提示、用户提示、工具描述、历史记录、记忆、工具输出，所有你给 LLM 以生成良好输出的内容。

Context window visualization

上下文窗口可视化

Figure 4. The context window — everything fed to the LLM including system prompt, user input, tool descriptions, and conversation history.

图 4. 上下文窗口 — 提供给 LLM 的所有内容，包括系统提示、用户输入、工具描述和对话历史。

Context Window is like the space inside LLM agent you can think as RAM and It is limited. And using that space(context window) effectively means Context Engineering.

上下文窗口就像 LLM 代理内部的空间，你可以将其视为 RAM，而且它是有限的。有效地使用这个空间（上下文窗口）就意味着上下文工程。

As you know that LLMs are just stateless functions right? You give them input, they give you output. That's it. To get good outputs, you need to give them good inputs. Sounds simple, right? But here's where it gets interesting.

你知道 LLM 只是无状态函数，对吧？你给它们输入，它们给你输出。就是这样。要获得好的输出，你需要给它们好的输入。听起来很简单，对吧？但这就是它变得有趣的地方。

You've probably seen the standard OpenAI message format:

你可能见过标准的 OpenAI 消息格式：

```json
[
  {"role": "system", "content": "You are..."},
  {"role": "user", "content": "Do something"},
  {"role": "assistant", "content": "..."},
  {"role": "tool", "content": "..."}
]
```

Which looks like this inside LLM Agent:

在 LLM Agent 内部看起来像这样：

Context structure inside LLM Agent

LLM Agent 内部的上下文结构

Figure 5. Message structure inside an LLM agent — system, user, assistant, and tool messages stacked in the context.

图 5. LLM 代理内部的消息结构 — 系统、用户、助手和工具消息堆叠在上下文中。

This works, sure. But is it optimal? Not at all and it will create noise cause not all information is important. You can pack information in a better way to avoid noise. The format is just a means to an end; what actually matters is giving the model the right information in a way it can use it.

这当然有效。但它是最优的吗？完全不是，而且它会产生噪音，因为不是所有信息都重要。你可以用更好的方式打包信息以避免噪音。格式只是达到目的的手段；真正重要的是以模型可以使用的方式给它正确的信息。

Why should you even care about context management?

你为什么要关心上下文管理？

So here is thing that even models with 1M context windows get lost way before hitting their limit. Like, really way way before...

所以事情是这样的，即使是拥有 1M 上下文窗口的模型也会在达到其极限之前就迷失。就像，真的远远在之前……

Means we have 150k to 200k context window length (in terms of tokens) to play where model can give its best performance so we need to use it very efficiently.

意味着我们有 150k 到 200k 的上下文窗口长度（以令牌计）可以使用，模型可以在这里提供最佳性能，所以我们需要非常有效地使用它。

Okay, but WHY is this happening?

好的，但为什么会发生这种情况？

You can skip this part ; Its not important to learn about this.

你可以跳过这部分；了解这个并不重要。

7. Mechanistic interpretability = trying to reverse engineer what's happening inside neural networks. Like opening up a brain and figuring out which neurons fire for what. It's a whole rabbit hole and honestly nobody fully understands it yet.

7. 机制可解释性 = 试图逆向工程神经网络内部发生的事情。就像打开大脑并弄清楚哪些神经元为什么而激活。这是一个完整的兔子洞，说实话，还没有人完全理解它。

To find the exact reason, we'd need to go deep into mechanistic interpretability7 of these models, which is way out of scope for this blog (and honestly, I don't fully understand it myself). But here's what I've learned from reading papers and building Hakken:

要找到确切原因，我们需要深入研究这些模型的机制可解释性7，这远远超出了这篇博客的范围（说实话，我自己也没有完全理解它）。但这是我从阅读论文和构建 Hakken 中学到的：

### 1. Error Compounding & Self-Conditioning

### 1. 错误复合和自我调节

So imagine you're solving a long math problem and you make a tiny mistake in step 3. Now every step after that is fucked. Same thing with LLMs.

想象一下你正在解决一个很长的数学问题，你在第 3 步犯了一个小错误。现在之后的每一步都搞砸了。LLM 也是一样。

Research shows models become more likely to make mistakes when their context contains errors from prior turns. One wrong tool call early on? The model sees that mistake, gets confused, makes another mistake, sees that mistake... you get the idea. This is especially very bad for long-horizon tasks where early mistakes distort everything that comes after.

研究表明，当模型的上下文包含先前轮次的错误时，模型更有可能犯错误。早期一个错误的工具调用？模型看到那个错误，感到困惑，犯另一个错误，看到那个错误……你明白的。这对于长时程任务尤其糟糕，其中早期错误会扭曲之后的所有内容。

But this one is interesting because this is not a common thing if you're using models from Anthropic or OpenAI; they are very good at recovering from mistakes. But if you're using other cheaper models, then you might face this issue a lot8.

但这个很有趣，因为如果你使用 Anthropic 或 OpenAI 的模型，这不是常见的事情；它们非常擅长从错误中恢复。但如果你使用其他更便宜的模型，那么你可能会经常遇到这个问题8。

8. I've seen this happen with Hakken on 50+ step tasks. One bad file edit early on and the agent keeps trying to fix something that was never broken.

8. 我在 Hakken 的 50+ 步任务中看到过这种情况。早期一个糟糕的文件编辑，代理会一直尝试修复从未损坏的东西。

### 2. Lost-in-the-Middle Effect

### 2. 迷失在中间效应

LLMs have this U-shaped attention pattern - they pay attention to stuff at the beginning and end, but ignore the middle. It's like when you're in a meeting and only remember what was said at the start and at the end. Same thing here.

LLM 有这种 U 形注意力模式 - 它们关注开头和结尾的内容，但忽略中间。就像你在会议中只记得开始和结束时说的话。这里也是一样的。

9. Chroma is the vector database company. They built this research to understand why RAG systems fail at scale.

9. Chroma 是向量数据库公司。他们建立了这项研究来了解为什么 RAG 系统在规模上失败。

Chroma did this research on context rot9, they tested 18 different LLMs (GPT-4.1, Claude 4, Gemini 2.5, Qwen3) and found that performance consistently degrades as input length increases, even on simple tasks. They called it context rot.

Chroma 对上下文腐烂进行了这项研究9，他们测试了 18 种不同的 LLM（GPT-4.1、Claude 4、Gemini 2.5、Qwen3），发现即使在简单任务上，性能也会随着输入长度的增加而持续下降。他们称之为上下文腐烂。

Context rot means when your model's performance drops as you add more tokens. You'd think more context = better performance, right? Nope. Wrong.

上下文腐烂意味着当你添加更多令牌时，模型的性能会下降。你会认为更多上下文 = 更好的性能，对吧？不对。错了。

Look at this experiment Chroma did:

看看 Chroma 做的这个实验：

Context rot performance degradation chart

上下文腐烂性能下降图表

Figure 6. Performance degradation across 18 models (GPT-4.1, Claude 4, Gemini 2.5, Qwen3). Blue = high-similarity, Red = low-similarity needle-question pairs. Accuracy drops significantly as context grows.

图 6. 18 个模型（GPT-4.1、Claude 4、Gemini 2.5、Qwen3）的性能下降。蓝色 = 高相似度，红色 = 低相似度的针-问题对。随着上下文增长，准确性显著下降。

Let's take a quick look into what's actually happening:

让我们快速了解一下实际发生的事情：

Models perform worse at 100k tokens than 10k tokens10 - same exact task, just more irrelevant stuff around it. Look at that graph; performance dropped.

模型在 100k 令牌时的表现比 10k 令牌时差10 - 完全相同的任务，只是周围有更多不相关的东西。看看那个图表；性能下降了。

10. We're talking 15-30% accuracy drop on some models. GPT-4.1 went from ~85% at 10k to ~60% at 100k on certain tasks. That's not a small difference, that's your agent becoming significantly dumber.

10. 我们谈论的是某些模型上 15-30% 的准确性下降。GPT-4.1 在某些任务上从 10k 时的 ~85% 下降到 100k 时的 ~60%。这不是一个小差异，这是你的代理变得明显更笨了。

Low similarity needle-question pairs tank at scale - when the question and answer aren't lexically similar, performance drops hard at longer contexts. That red line in the graph? Yeah, that's not good.

低相似度针-问题对在规模上表现很差 - 当问题和答案在词汇上不相似时，在更长的上下文中性能会大幅下降。图表中的那条红线？是的，这不好。

Distractors have non-uniform impact - kinda related but wrong info confuses models more as context grows. Some distractors make it way worse than others.

干扰因素具有非均匀影响 - 有点相关但错误的信息会随着上下文的增长而更多地混淆模型。一些干扰因素比其他干扰因素更糟糕。

**Structured text can actually cause problems** - Models perform better on shuffled random text than coherent logical text. This suggests that coherent structure creates attention patterns that interfere with retrieval. Also cause model are great at mimicking.

结构化文本实际上可能会导致问题 - 模型在打乱的随机文本上的表现比连贯的逻辑文本更好。这表明连贯的结构会创建干扰检索的注意力模式。而且因为模型非常擅长模仿。

Chroma's conclusion: "Even the most capable models are sensitive to how information is presented in context." And this isn't getting fixed by better models. GPT-5.1 and Claude 4 still suffer from this.

Chroma 的结论："即使是最有能力的模型也对上下文中信息的呈现方式敏感。"而且这不会通过更好的模型得到修复。GPT-5.1 和 Claude 4 仍然遭受这个问题。

So it means it's not just about whether relevant information is present in your model's context. What matters more is what information you include and how you present it.

所以这意味着这不仅仅是关于相关信息是否存在于你的模型的上下文中。更重要的是你包含什么信息以及如何呈现它。

The takeaway? Your job is to pack the right information into 100k-200k tokens where models perform best. That's your sweet spot. And more importantly - keep your agent's context clean. Every error, every irrelevant tool output.

要点？你的工作是将正确的信息打包到 100k-200k 令牌中，模型在这里表现最好。这是你的最佳点。更重要的是 - 保持你的代理的上下文干净。每个错误，每个不相关的工具输出。

Many AI labs are working on solving long horizon problem with RL or some other memory kinda thing but these models are still incredibly useful - you just need to own every single token the model gets to see.

许多 AI 实验室正在通过 RL 或其他一些记忆类的东西来解决长时程问题，但这些模型仍然非常有用 - 你只需要拥有模型看到的每一个令牌。

So yeah, context engineering is everything!

所以是的，上下文工程就是一切！

### Lets see some context engineering example that I did in Hakken

### 让我们看看我在 Hakken 中做的一些上下文工程示例

#### 1. Simple System Prompt

#### 1. 简单的系统提示

So here is the thing if you're working with cheaper or free model from openrouter then you're going to have very hard time.

所以事情是这样的，如果你使用来自 openrouter 的更便宜或免费的模型，那么你将会很难受。

WHY? Cause sometimes small and minimal prompt works and sometimes it doesn't. So I did not want to spend too much money on this project so I used those cheaper models with huge ass system prompt with all those xml tags and all that shit. I used to switch between cheaper model and cluade according to need.

为什么？因为有时小而简单的提示有效，有时则不然。所以我不想在这个项目上花太多钱，所以我使用了那些更便宜的模型，带有巨大的系统提示，包含所有那些 xml 标签和所有那些东西。我过去根据需要在更便宜的模型和 claude 之间切换。

But if you are working with good model like Claude Opus-4.5 you just give minimal system prompt and it will work fine.

但如果你使用像 Claude Opus-4.5 这样的好模型，你只需给出最小的系统提示，它就会工作得很好。

For Example :

例如：

```python
system_prompt = """
You are an expert coding assistant. You help users with coding tasks by reading files, executing commands, editing code, and writing new files.

Available tools:
- read: Read file contents
- bash: Execute bash commands
- edit: Make surgical edits to files
- write: Create or overwrite files

Guidelines:
- Use bash for file operations like ls, grep, find
- Use read to examine files before editing
- Use edit for precise changes (old text must match exactly)
- Use write only for new files or complete rewrites
- When summarizing your actions, output plain text directly - do NOT use cat or bash to display what you did
- Be concise in your responses
- Show file paths clearly when working with files

Documentation:
- Your own documentation (including custom model setup and theme creation) is at: /path/to/README.md
- Read it when users ask about features, configuration, or setup, and especially if the user asks you to add a custom model or provider, or create a custom theme.
```

#### 2. Simple Tools Section

#### 2. 简单的工具部分

In the beginning I wanted all tools that this world has to offer. Cause I heard somewhere your agent is better if it has all tools to offer. But then I realized that's not the case. Cause most of the time you don't need all tools to offer. You only need the tools that are relevant to the task.

一开始我想要这个世界所能提供的所有工具。因为我在某处听说，如果你的代理有所有可用的工具，它会更好。但后来我意识到事实并非如此。因为大多数时候你不需要提供所有工具。你只需要与任务相关的工具。

So I decided to give only the tools that are relevant to the task. And it worked fine.

所以我决定只提供与任务相关的工具。而且效果很好。

You can let your agent use bash command for most of the tasks like for file operations like ls, grep, find. By doing this you can save huge number of tokens.

你可以让你的代理使用 bash 命令来完成大多数任务，比如文件操作，如 ls、grep、find。通过这样做，你可以节省大量令牌。

Hakken do not have web search tool. It used to have but it is not that useful. I think better way is to find the information by yourself that you think are important and save in .md docs and provide directly it to agent's context window.

Hakken 没有网络搜索工具。它曾经有，但它不是那么有用。我认为更好的方法是自己找到你认为重要的信息，保存在 .md 文档中，并直接提供给代理的上下文窗口。

#### 3. Compression with Structure

#### 3. 结构化压缩

When my context hits 80%, I summarize with an LLM. But I don't just free-form it - I use a schema to make sure I keep what matters:

当我的上下文达到 80% 时，我用 LLM 进行总结。但我不只是自由形式 - 我使用模式来确保保留重要的内容：

```python
def manage_context(self):
    if self.get_context_usage() >= 0.8:
        old_messages = self.messages[2:-5]

        summary_prompt = """Summarize preserving:
        - Key decisions made
        - Errors encountered and solutions
        - Pending todos
        Keep under 500 tokens."""

        summary = self.llm.generate(old_messages + [summary_prompt])

        self.messages = [
            self.messages[0],  # System prompt
            {"role": "assistant", "content": f"[Summary]\n{summary}"},
            *self.messages[-5:]  # Last 5 messages
        ]
```

Does this actually work? Yeah. Compression at 80% reduced my average context usage by 35-40%11. But you need to be careful about what to preserve - decisions, errors, and todos are critical.

这真的有效吗？是的。在 80% 时压缩将我的平均上下文使用量减少了 35-40%11。但你需要小心保留什么 - 决策、错误和待办事项是关键的。

11. When NOT to compress: if you're doing code review or debugging, you need the full context. Compression kills nuance. Also don't compress if you're under 50k tokens - just let it ride.

11. 什么时候不压缩：如果你在做代码审查或调试，你需要完整的上下文。压缩会破坏细微差别。此外，如果你低于 50k 令牌，也不要压缩 - 让它自然进行。

Here you can use other cheaper model cause most of the models are good at summarisation task. And also this is not super important. I think it's better to start new session if it comes to stage of summarisation.

在这里你可以使用其他更便宜的模型，因为大多数模型都擅长总结任务。而且这也不是超级重要。我认为如果到了总结阶段，最好开始新会话。

#### 4. Aggressive Tool Result Management

#### 4. 激进的工具结果管理

Tool outputs are massive. Like, a file read can dump 1000 lines into your context. And sometimes not all of that is important. Here's what I do - I automatically clear old tool results after every 10 tool calls (keeping the last 5). Anthropic actually launched this as a platform feature12. It's that important.

工具输出是巨大的。比如，一个文件读取可以将 1000 行转储到你的上下文中。有时并非所有这些都很重要。这是我的做法 - 我在每 10 次工具调用后自动清除旧的工具结果（保留最后 5 个）。Anthropic 实际上将此作为平台功能推出12。它就是那么重要。

12. Anthropic added "tool result caching" in late 2025. You can now mark tool results as ephemeral and they handle the cleanup.

12. Anthropic 在 2025 年末添加了"工具结果缓存"。你现在可以将工具结果标记为临时的，它们会处理清理。

```python
def clear_old_tool_results(messages: list, keep_last: int = 5):
    """Clear tool results older than last N tool calls"""
    tool_indices = [
        i for i, msg in enumerate(messages)
        if msg.get("role") == "tool"
    ]

    if len(tool_indices) <= keep_last:
        return messages

    # Keep system prompt + user messages + last N tool results
    indices_to_clear = tool_indices[:-keep_last]

    for idx in indices_to_clear:
        messages[idx]["content"] = "[Result cleared - see recent outputs]"

    return messages

# Call this periodically in your agent loop
if tool_call_count % 10 == 0:
    messages = clear_old_tool_results(messages)
```

Why? Because once a tool has been called deep in history, why would the agent need to see the raw result again? You're not losing information, you're just cleaning up stuff that's not needed anymore.

为什么？因为一旦工具在历史深处被调用，代理为什么还需要再次看到原始结果？你没有丢失信息，你只是清理不再需要的东西。

#### 5. KV-Cache Optimization

#### 5. KV-Cache 优化

13. Manus is that AI agent company from China that went viral. They published a blog about their architecture and KV-cache optimization was their #1 production metric. If they're optimizing for it, you should too.

13. Manus 是那家来自中国的走红的 AI 代理公司。他们发布了一篇关于他们架构的博客，KV-cache 优化是他们的第一大生产指标。如果他们正在优化它，你也应该这样做。

If you pick one metric for production agents, pick this: KV-cache hit rate - Manus13. Why? Because cached tokens on Sonnet-4.5 cost $0.30/MTok vs $3/MTok uncached. That's a 10x cost reduction right there and directly affect latency too.

如果你为生产代理选择一个指标，选择这个：KV-cache 命中率 - Manus13。为什么？因为 Sonnet-4.5 上的缓存令牌成本为 $0.30/MTok，而未缓存的为 $3/MTok。这直接减少了 10 倍的成本，也直接影响延迟。

But here's the thing - you need to follow three rules to actually make KV-cache work for you:

但事情是这样的 - 你需要遵循三个规则才能真正让 KV-cache 为你工作：

Keep your prompt prefix stable. If even one token changes in your prompt, the entire KV-cache from that point onward is invalidated. The model has to recompute everything after that token.

保持你的提示前缀稳定。如果你的提示中即使一个令牌发生变化，从那时起的整个 KV-cache 都将失效。模型必须重新计算该令牌之后的所有内容。

Here's what NOT to do:

这是不该做的：

```python
system_prompt = f"""
Current time: {datetime.now().isoformat()}
You are a helpful assistant...
"""
```

Every single request gets a different timestamp → entire cache misses → you're paying full price for every request.

每个请求都会得到不同的时间戳 → 整个缓存未命中 → 你为每个请求支付全价。

How prompt caching work overview

提示缓存工作概述

```
Request 1:
[System prompt: 10,000 tokens about coding guidelines]
User: "Write a function to sort an array"
→ Processes everything, caches the system prompt

Request 2 (minutes later):
[Same system prompt: 10,000 tokens]
User: "Write a function to reverse a string"
→ Retrieves cached system prompt instantly
→ Only processes the new user message
```

Make context append-only. Don't go back and modify previous messages in your context. The moment you edit something in the middle of your history, you break the cache from that point forward. Instead of updating old messages, append new information. Yes, this uses slightly more tokens. But it keeps your cache valid, which saves you way more in the long run.

使上下文仅追加。不要回去修改上下文中的先前消息。当你在历史记录中间编辑某些内容时，你会从那时起破坏缓存。而不是更新旧消息，追加新信息。是的，这会使用稍微多一点的令牌。但它使你的缓存保持有效，从长远来看可以为你节省更多。

Deterministic serialization. Many languages don't guarantee stable JSON key ordering. The same data might serialize differently across requests, which breaks caching. Always sort your JSON keys:

确定性序列化。许多语言不保证稳定的 JSON 键排序。相同的数据可能在不同请求中序列化不同，这会破坏缓存。始终对你的 JSON 键进行排序：

```python
# Always sort keys to get consistent output
def serialize_for_cache(data):
    return json.dumps(data, sort_keys=True)
```

#### 6. Structured Note-Taking

#### 6. 结构化笔记

This is one of my favorite context optimizations. Instead of keeping everything in context, let the agent write notes to disk and retrieve them later.

这是我最喜欢的上下文优化之一。与其将所有内容保留在上下文中，不如让代理将笔记写入磁盘并稍后检索它们。

Structured note-taking with todo.md

使用 todo.md 进行结构化笔记

Figure 7. Structured note-taking in Hakken — the agent maintains a todo.md file to persist context outside the token window.

图 7. Hakken 中的结构化笔记 — 代理维护一个 todo.md 文件以在令牌窗口之外持久化上下文。

In Hakken, when tasks get complex, the agent creates a todo.md file and updates it as work progresses. This serves two purposes:

在 Hakken 中，当任务变得复杂时，代理会创建一个 todo.md 文件并在工作进展时更新它。这有两个目的：

First, persistent memory - critical context lives outside the context window. No token limits here.

首先，持久性记忆 - 关键上下文存在于上下文窗口之外。这里没有令牌限制。

Second, attention manipulation - by constantly rewriting the todo list, the agent "recites" its objectives into the end of context, pushing goals into recent attention span. Remember that U-shaped attention pattern? This exploits it. The model keeps seeing its main goal at every turn of the conversation, so it doesn't forget what it's supposed to be doing.

其次，注意力操纵 - 通过不断重写待办事项列表，代理将其目标"背诵"到上下文的末尾，将目标推入最近的注意力范围。还记得那个 U 形注意力模式吗？这利用了它。模型在对话的每一轮都看到它的主要目标，所以它不会忘记它应该做什么。

In Some Tasks I found that agent work better without this in built todo tool. I think it breaks the flow of agent.

在某些任务中，我发现没有这个内置的待办事项工具，代理工作得更好。我认为它打破了代理的流程。

#### 7. Progressive Disclosure

#### 7. 渐进式披露

Don't pre-load everything. Use just-in-time retrieval. Instead of embedding everything and retrieving before inference, maintain lightweight identifiers (file paths, URLs, etc.) and dynamically load data at runtime using tools.

不要预加载所有内容。使用即时检索。与其嵌入所有内容并在推理之前检索，不如维护轻量级标识符（文件路径、URL 等）并在运行时使用工具动态加载数据。

```python
# Don't do this
context = load_all_docs() + load_all_files()

# Do this
context = {"file_paths": [...], "available_tools": [...]}
# Let agent explore and load what it needs
```

This actually mirrors human cognition pretty well. We don't memorize entire corpora, we use filing systems, inboxes, bookmarks to retrieve on demand.

这实际上很好地反映了人类的认知。我们不会记住整个语料库，我们使用归档系统、收件箱、书签来按需检索。

#### 8. Short Sessions, Not Long

#### 8. 短会话，而非长会话

Amp's research is clear on this: 200k tokens is plenty. Long threads they mess up, fall over, vomit all over you (okay maybe I'm exaggerating, but you get the point).

Amp 的研究对此很明确：200k 令牌已经足够。长线程会搞砸，倒下，向你吐得到处都是（好吧，也许我夸张了，但你明白了）。

Break work into small threads. One task per thread. Use thread references to carry context forward:

将工作分解成小线程。每个线程一个任务。使用线程引用将上下文向前传递：

```python
# Thread 1: Basic implementation
# Thread 2: Refactor using read_thread(thread_1_id)
# Thread 3: Tests using read_thread(thread_2_id)
```

Each thread stays focused. Total work might use 1M tokens across threads, but each individual thread stays sharp under 200k.

每个线程保持专注。总工作可能跨线程使用 1M 令牌，但每个单独的线程在 200k 以下保持敏锐。

### Own Your Prompts and Control Flow

### 掌控你的提示和控制流

Alright, let's talk about something that drives me crazy. I see so many frameworks where you do something like this:

好吧，让我们谈谈让我抓狂的事情。我看到很多框架，你会这样做：

```python
agent = Agent(role="...", goal="...", personality="...")
```

Hidden prompts in agent frameworks

代理框架中的隐藏提示

Figure 8. From Hamel Husain's blog on prompt ownership — many frameworks hide what actually gets sent to the model.

图 8. 来自 Hamel Husain 关于提示所有权的博客 — 许多框架隐藏了实际发送给模型的内容。

And you have no idea what tokens are actually going to the model. Maybe it's great prompt engineering, or maybe it's sending some bullshit system prompt that you can't control or even see. You won't know until your agent starts giving you random stuff and then you'll realize you have zero control.

你不知道实际上什么令牌会发送给模型。也许它是很好的提示工程，或者也许它发送的是一些你无法控制甚至看不到的废话系统提示。你不会知道，直到你的代理开始给你随机的东西，然后你会意识到你没有任何控制权。

Not only that, but extracting the final prompt that gets sent to the model from those frameworks is way too hard.

不仅如此，从那些框架中提取发送给模型的最终提示也太难了。

In Hakken, **I own every single token that goes into the context window:**

在 Hakken 中，我拥有进入上下文窗口的每一个令牌：

```python
def get_system_prompt():
    """
    You are Hakken, an autonomous coding agent.

    Core behaviors:
    - Read files before editing
    - Write clean, idiomatic code
    - Explain your reasoning

    Available tools: [list with descriptions]
    Constraints: [safety rules]
    """
```

Is my prompt perfect? Fuck no14. But I can iterate on it easily because I know exactly what the model is seeing.

我的提示完美吗？当然不是14。但我可以轻松地迭代它，因为我确切地知道模型看到的是什么。

14. The beauty of iteration: my first prompt was 50 lines of garbage. Now it's 20 lines of slightly less garbage. But each version came from seeing what actually broke and fixing it. Can't do that with a black box.

14. 迭代的美妙之处：我的第一个提示是 50 行垃圾。现在它是 20 行稍微少一点的垃圾。但每个版本都来自于看到实际上出了什么问题并修复它。用黑盒做不到这一点。

When something breaks, it's easy to debug. I can change how the model should act. The benefit here is full control.

当出现问题时，很容易调试。我可以改变模型应该如何行动。这里的好处是完全控制。

Same thing with control flow. Own your loop. Most agent frameworks give you this black-box loop where you feed in your task and hope it works. Build your own loop that you actually understand and can modify when shit goes sideways:

控制流也是一样。掌控你的循环。大多数代理框架给你这个黑盒循环，你输入任务并希望它有效。构建你自己实际理解并可以在出现问题时修改的循环：

```python
async def _recursive_message_handling(self):
    # 1. Compress context if needed
    self._history_manager.auto_messages_compression()

    # 2. Call LLM
    request = self._build_api_request()
    stream = self._api_client.get_completion_stream(request)
    response = self._response_handler.process_stream(stream)

    # 3. Add to history
    self.add_message(response)

    # 4. Tool calls? Execute and recurse
    if has_tool_calls(response):
        await self._tool_executor.handle_tool_calls(response.tool_calls)
        await self._recursive_message_handling()  # Loop again
    else:
        await self._handle_conversation_turn(response)
```

Why own the loop? **So you can break out when shit happens:**

为什么要掌控循环？这样当出现问题时你可以退出：

```python
if tool.is_dangerous():
    approval = await ask_human(f"Agent wants to {tool.name}. Allow?")
    if not approval:
        return "Task cancelled"
```

Can't do this if a framework owns your control flow. TBH it's not hard to build, it's just a recursive loop.

如果框架拥有你的控制流，就无法做到这一点。说实话，构建起来并不难，它只是一个递归循环。

I added this ask user before using tool becuase I was working with cheaper model But I think You don't need this if working with models like Opus-4.5. You can just mentioned your security stuff in system prompt.

我添加了在使用工具之前询问用户，因为我在使用更便宜的模型，但我认为如果使用像 Opus-4.5 这样的模型，你不需要这个。你可以只在系统提示中提到你的安全问题。

### Running Agent On Trust Mode

### 在信任模式下运行代理

So I have given all the permission to hakken agent at the start with like asking permission to run this and run or ask question. **But when I gave complete permission it started giving me better results**. Idk is this a thing or not but yeah this is what I observed.

所以我一开始给了 hakken 代理所有权限，就像请求运行这个和运行或提问的权限。但当我给予完全权限时，它开始给我更好的结果。不知道这是不是一回事，但是的，这是我观察到的。

Then to find out the reason and it was simple reason: that is it breaks the thinking process and when agent sees that it can ask for permission it behaves like it has less confidence in it.

然后找出原因，这是一个简单的原因：就是它打破了思考过程，当代理看到它可以请求权限时，它表现得好像对自己缺乏信心。

So basically when you keep asking agent for permission every single step, it loses the context and has to restart its thinking chain again and again. It's like trying to solve a puzzle but someone keeps interrupting you after every piece - you lose track of the bigger picture.

所以基本上当你每一步都要求代理请求权限时，它会失去上下文，不得不一次又一次地重新启动其思维链。就像试图解决一个谜题，但有人在每一块之后一直打断你 - 你会失去对大局的把握。

Also I noticed that when agent knows it has full permission upfront, it plans better. Like it can think multiple steps ahead instead of just focusing on one step at a time. It's more strategic about the whole goal instead of being stuck in permission-asking loop.

此外，我注意到当代理知道它预先拥有完全权限时，它计划得更好。就像它可以提前思考多个步骤，而不是一次只专注于一个步骤。它对整个目标更具战略性，而不是陷入请求权限的循环中。

The downside is obviously you lose some control and oversight. But for complex tasks where you trust the agent, giving blanket permission upfront just works way better. It's similar to how you'd work with a human assistant - if you trust them, you give them authority to make decisions instead of micromanaging every small thing.

缺点显然是你失去了一些控制和监督。但对于你信任代理的复杂任务，预先给予全面权限效果更好。这类似于你如何与人类助手合作 - 如果你信任他们，你给他们权力做决定，而不是微观管理每一件小事。

Note : This work really great with Claude Opus 4.5.

注意：这在 Claude Opus 4.5 上效果非常好。

## Evaluation: Build It First

## 评估：先构建它

Okay, so agent eval is not like normal LLM eval. Agents are complex cause they reason, use tools, make plans, work across multiple turns.

好的，所以代理评估不像正常的 LLM 评估。代理很复杂，因为它们推理、使用工具、制定计划、跨多轮工作。

Structured note-taking with todo.md

使用 todo.md 进行结构化笔记

Figure 9. Complex Evaluation of LLM Agent — the agent have so many components and you need to evaluate separately.

图 9. LLM Agent 的复杂评估 — 代理有这么多组件，你需要单独评估。

You know traditional testing expects same input → same output but agents are non-deterministic right? Cause they operate across multi-step workflows, make real API calls, and maintain context.

你知道传统测试期望相同的输入 → 相同的输出，但代理是非确定性的，对吧？因为它们在多步工作流程中操作，进行真实的 API 调用，并维护上下文。

So you need completely different approach. You need to break down the components of the agents and test it separately. I started with very simple evals like first save all traces of of your agent. And just note down failure and add it your eval datasets.

所以你需要完全不同的方法。你需要分解代理的组件并单独测试它。我从非常简单的评估开始，比如首先保存代理的所有跟踪。然后只需记录失败并将其添加到你的评估数据集中。

**Components That Need Evaluation**

**需要评估的组件**

- Retriever(s) - Are they pulling relevant docs? (Contextual Relevancy)
- Reranker - Is it reordering results correctly? (Ranking quality)
- LLM - Is the output relevant and faithful? (Answer Relevancy, Faithfulness)
- Tool Calls - Right tools? Right params? Efficient? (Tool Correctness, Tool Efficiency)
- Planning Module - Is the plan logical and complete? (Plan Quality)
- Reasoning Steps - Is the thinking coherent and relevant? (Reasoning Quality)
- Sub-agents - Are they completing their subtasks? (Task Completion per agent)
- Router/Orchestrator - Is it routing to the right component? (Routing Accuracy)
- Memory System - Is it storing/retrieving relevant info? (Memory Relevancy)
- Final Answer - Did it complete the task? (Task Completion via G-Eval)
- Safety Check - Any toxic/biased/harmful content? (Safety Metrics)
- Full Pipeline - Did the entire agent workflow succeed? (Overall Task Completion)

- 检索器 - 它们是否检索到相关文档？（上下文相关性）
- 重排器 - 它是否正确地重新排序结果？（排名质量）
- LLM - 输出是否相关且忠实？（答案相关性、忠实度）
- 工具调用 - 正确的工具？正确的参数？高效？（工具正确性、工具效率）
- 规划模块 - 计划是否合乎逻辑且完整？（计划质量）
- 推理步骤 - 思考是否连贯且相关？（推理质量）
- 子代理 - 它们是否完成了子任务？（每个代理的任务完成情况）
- 路由器/编排器 - 它是否路由到正确的组件？（路由准确性）
- 记忆系统 - 它是否存储/检索相关信息？（记忆相关性）
- 最终答案 - 它是否完成了任务？（通过 G-Eval 的任务完成情况）
- 安全检查 - 有任何有毒/有偏见/有害的内容吗？（安全指标）
- 完整管道 - 整个代理工作流程是否成功？（总体任务完成情况）

And also one thing I tried is building monitoring UI where you can monitor all the agent traces with all events. And also monitor cost and latency which becomes useful to spot the failure of your agent and add those data to your evals.

我还尝试的一件事是构建监控 UI，你可以在其中监控所有代理跟踪及所有事件。还可以监控成本和延迟，这对于发现代理的失败并将这些数据添加到你的评估中很有用。

Use LLM for most of the evals cause this is complex task so binary metrics would not give you much result.

对大多数评估使用 LLM，因为这是复杂的任务，所以二进制指标不会给你太多结果。

Evaluation section is kinda weak cause personally I built Hakken just for fun toy project. So I did not care about benchmarks. I was like as long as it works, it works.

评估部分有点弱，因为我个人构建 Hakken 只是为了有趣的玩具项目。所以我不关心基准测试。我就像只要它工作，它就工作。

## What About Memory?

## 记忆呢？

Let's understand first why we need memory. Simply, we know that LLMs are stateless; they don't remember anything.

让我们首先了解为什么我们需要记忆。简单地说，我们知道 LLM 是无状态的；它们不记得任何东西。

So every new conversation you start with LLM is from scratch.

所以你与 LLM 开始的每一次新对话都是从头开始的。

Suppose you're starting new session want to continue old task then you need to give complete context of old incomplete task to the model.

假设你正在开始新会话，想继续旧任务，那么你需要给模型旧的未完成任务的完整上下文。

Okay you get the point right? That agent does not remember any stuff.

好的，你明白了吧？代理不记得任何东西。

Suppose you are working on a coding problem and you want your output in a specific format every time you perform a coding task with an agent. So what can you do here? You can mention how you want your output every time you give input. But what if the agent could figure out your preferred format and remember it? That's where you need Memory.

假设你正在处理编码问题，并且每次使用代理执行编码任务时，你都希望输出采用特定格式。那么你在这里可以做什么？你可以每次给输入时提到你想要的输出方式。但如果代理可以弄清楚你偏好的格式并记住它呢？这就是你需要记忆的地方。

You will see so many people talking about memory this and memory that. The most important thing to build a reliable agent is context engineering. That's it.

你会看到很多人谈论记忆这个和记忆那个。构建可靠代理最重要的事情是上下文工程。就是这样。

Okay then memory is not needed? It's needed but not everywhere. Let me explain with examples.

好的，那么记忆不需要？它是需要的，但不是到处都需要。让我用例子解释。

### When Memory is NOT Needed (Horizontal Applications)

### 何时不需要记忆（水平应用）

Think about ChatGPT or Claude or any general purpose chatbot. Every conversation is different right? Today someone asks about cooking recipes, tomorrow someone asks about fixing their car, next person asks about math problems.

想想 ChatGPT 或 Claude 或任何通用聊天机器人。每次对话都不同，对吧？今天有人问烹饪食谱，明天有人问修车，下一个人问数学问题。

What's the point of remembering stuff here? Each conversation is fresh, new context, new problem. You just need good prompts and proper context in that moment. That's why you don't see ChatGPT remembering "oh last time this user liked short answers so let me give short answers every time". It doesn't make sense.

在这里记住东西有什么意义？每次对话都是新鲜的，新的上下文，新的问题。你只需要在那一刻有好的提示和适当的上下文。这就是为什么你看不到 ChatGPT 记住"哦，上次这个用户喜欢简短的答案，所以让我每次都给简短的答案"。这没有意义。

Same thing if you building customer support bot for different types of queries, or building general Q&A system. Memory just adds complexity for no reason. Good context engineering in that moment is enough.

如果你为不同类型的查询构建客户支持机器人，或构建通用问答系统，也是一样。记忆只是无缘无故地增加了复杂性。在那一刻良好的上下文工程就足够了。

### When Memory IS Needed (Vertical Applications)

### 何时需要记忆（垂直应用）

But now think about coding agent. **You are working on the same codebase for weeks or months.** The agent should remember your style preferences right? Like you told it once "hey don't write too many comments, follow clean code guidelines, don't add unnecessary error handling everywhere, keep functions small".

但现在想想编码代理。你在同一个代码库上工作数周或数月。代理应该记住你的风格偏好，对吧？就像你告诉它一次"嘿，不要写太多注释，遵循干净的代码指南，不要到处添加不必要的错误处理，保持函数小巧"。

Without memory, you will keep repeating same thing again and again in every conversation. That's annoying. Here memory is goldmine because the domain is same, the task is specific, the context is related.

没有记忆，你将在每次对话中一遍又一遍地重复同样的事情。这很烦人。在这里记忆是金矿，因为领域相同，任务特定，上下文相关。

Another example - let's say you're building a personal finance agent. It should remember "user wants aggressive investment strategy" or "user has low risk tolerance" or "user prefers index funds". Because every conversation is about the same thing - their money, their portfolio. Memory makes sense here.

另一个例子 - 假设你正在构建个人财务代理。它应该记住"用户想要激进的投资策略"或"用户风险承受能力低"或"用户更喜欢指数基金"。因为每次对话都是关于同样的事情 - 他们的钱，他们的投资组合。在这里记忆是有意义的。

Or think about email writing agent for sales team. It should remember company tone, writing style, what kind of emails convert better, etc. Because it's doing the same task again and again.

或者想想销售团队的电子邮件写作代理。它应该记住公司语气、写作风格、什么样的电子邮件转换更好等。因为它一遍又一遍地做同样的任务。

**Conclusion:** when building vertical LLM application for solving very specific task then memory might be useful. Like coding agents, personal assistants, specialized tools etc. But if you are building horizontal LLM application where every query is different then it's good to have but not a must. Focus on context engineering first, add memory only if it actually solves a real problem.

**结论：**当构建用于解决非常特定任务的垂直 LLM 应用时，记忆可能很有用。比如编码代理、个人助手、专业工具等。但如果你正在构建水平 LLM 应用，其中每个查询都不同，那么有它很好但不是必须的。首先关注上下文工程，仅在它实际解决真实问题时添加记忆。

Here's a simple memory implementation for a coding agent:

这是编码代理的简单记忆实现：

```python
import json
from pathlib import Path

class AgentMemory:
    def __init__(self, memory_file: str = ".hakken/memory.json"):
        self.memory_file = Path(memory_file)
        self.memory = self._load()

    def _load(self) -> dict:
        if self.memory_file.exists():
            return json.loads(self.memory_file.read_text())
        return {"preferences": {}, "learnings": []}

    def save(self):
        self.memory_file.parent.mkdir(exist_ok=True)
        self.memory_file.write_text(json.dumps(self.memory, indent=2))

    def add_preference(self, key: str, value: str):
        """Store user preferences like coding style"""
        self.memory["preferences"][key] = value
        self.save()

    def add_learning(self, learning: str):
        """Store things agent learned about the codebase"""
        self.memory["learnings"].append(learning)
        self.save()

    def get_context(self) -> str:
        """Format memory for injection into system prompt"""
        if not self.memory["preferences"] and not self.memory["learnings"]:
            return ""

        context = "\n## User Preferences (from past sessions)\n"
        for key, value in self.memory["preferences"].items():
            context += f"- {key}: {value}\n"

        if self.memory["learnings"]:
            context += "\n## Codebase Knowledge\n"
            for learning in self.memory["learnings"][-10:]:  # Last 10
                context += f"- {learning}\n"

        return context

# Usage in agent
memory = AgentMemory()
system_prompt = BASE_PROMPT + memory.get_context()
```

Simple file-based storage. No vector DB needed for most use cases. The key is knowing when to store something - I trigger memory updates when the agent detects explicit preferences ("always use type hints") or learns something about the codebase structure.

简单的基于文件的存储。大多数用例不需要向量数据库。关键是知道何时存储某些内容 - 当代理检测到明确的偏好（"始终使用类型提示"）或了解有关代码库结构的某些内容时，我会触发记忆更新。

## The Subagent Pattern

## 子代理模式

Subagents are very useful. They can help you reduce the context window load from the main agent by completing tasks in isolated context window.

子代理非常有用。它们可以通过在隔离的上下文窗口中完成任务来帮助你减少主代理的上下文窗口负载。

Structured note-taking with todo.md

使用 todo.md 进行结构化笔记

Figure 10. Parallel interdependent tasks in subagents.

图 10. 子代理中的并行相互依赖任务。

In Hakken, I have general-purpose, code-review, test-writer, and refactor subagents. Each one has its own specialized prompt.

在 Hakken 中，我有通用、代码审查、测试编写和重构子代理。每个都有自己的专门提示。

Here's a simple subagent pattern:

这是一个简单的子代理模式：

```python
class SubAgent:
    def __init__(self, name: str, system_prompt: str):
        self.name = name
        self.system_prompt = system_prompt
        self.messages = []  # Fresh context for each subagent

    async def run(self, task: str) -> str:
        """Run task in isolated context"""
        self.messages = [
            {"role": "system", "content": self.system_prompt},
            {"role": "user", "content": task}
        ]
        return await agent_loop(self.messages)

# Define specialized subagents
code_reviewer = SubAgent(
    name="code-review",
    system_prompt="""You are a code reviewer.
    Focus on: bugs, security issues, performance.
    Output format: JSON with issues array."""
)

test_writer = SubAgent(
    name="test-writer",
    system_prompt="""You are a test writer.
    Write comprehensive unit tests.
    Use pytest. Cover edge cases."""
)

# Main agent delegates to subagents
async def main_agent(task: str):
    if "review" in task.lower():
        return await code_reviewer.run(task)
    elif "test" in task.lower():
        return await test_writer.run(task)
    else:
        return await general_agent.run(task)
```

When subagents DON'T work: Parallel interdependent tasks. If your main agent is building a game engine while a subagent is designing a bird sprite, integration fails; they don't communicate with each other during the work.

子代理不起作用的情况：并行相互依赖的任务。如果你的主代理正在构建游戏引擎，而子代理正在设计鸟的精灵，集成会失败；它们在工作期间不会相互**通信**。

There are ways to use subagent for the same task by sharing the context between main and subagent which I think is too complex and can degrade the agent performance. So I believe in keeping it simple.

有一些方法可以通过在主代理和子代理之间共享上下文来为同一任务使用子代理，但我认为这太复杂了，可能会降低代理的性能。所以我相信保持简单。

Use subagents for deep research (parallel exploration), code review (non-interdependent), and sequential tasks (one completes, next starts with results).

将子代理用于深入研究（并行探索）、代码审查（非相互依赖）和顺序任务（一个完成，下一个使用结果开始）。

## The Simple Stuff That Matters

## 重要的简单事项

Let me share a few simple patterns that make a real difference:

让我分享一些真正有影响的简单模式：

### Compact Errors

### 紧凑错误

Tools fail constantly. You need to add errors to context so the LLM can see what broke, but full stack traces are massive:

工具经常失败。你需要将错误添加到上下文中，以便 LLM 可以看到什么出错了，但完整的堆栈跟踪非常庞大：

```python
def _compact_error(self, error: str) -> str:
    if len(error) <= 800:
        return error

    lines = error.strip().split('\n')
    if len(lines) <= 6:
        return error[:800]

    head = '\n'.join(lines[:2])
    tail = '\n'.join(lines[-3:])
    omitted = len(lines) - 5
    return f"{head}\n[...{omitted} lines omitted...]\n{tail}"
```

And track consecutive errors. If the agent fails 3 times in a row on the same thing, break out - it clearly can't fix it.

并跟踪连续错误。如果代理在同一件事上连续失败 3 次，就退出 - 它显然无法修复它。

### Skills > MCPs

### Skills > MCPs

MCPs (Model Context Protocol) consume tens of thousands of tokens. Instead, You use Skills - they're just Markdown files with YAML metadata. Super simple:

MCP（模型上下文协议）消耗数万个令牌。相反，你使用 Skills - 它们只是带有 YAML 元数据的 Markdown 文件。超级简单：

```markdown
---
name: pdf-creator
description: Creates PDFs from markdown
applies_when: User requests PDF output
---

# PDF Creation Skill

To create a PDF:
1. Use markdown-pdf library
2. Validate output with check_pdf_size()
3. Save to /mnt/outputs/
```

Models can read the skill metadata (few dozen tokens), then load full details only when needed. Progressive disclosure keeps context clean.

模型可以读取技能元数据（几十个令牌），然后仅在需要时加载完整细节。渐进式披露保持上下文干净。

## What Tech Stack To Use

## 使用什么技术栈

Before building Hakken I went through some repos like OpenCode and Codex just to understand what they are using to build this. So Codex used Rust to build their agent and OpenCode uses TypeScript.

在构建 Hakken 之前，我浏览了一些仓库，如 OpenCode 和 Codex，只是为了了解他们使用什么来构建这个。所以 Codex 使用 Rust 构建他们的代理，OpenCode 使用 TypeScript。

There are benefits of using both but I think it's better to stick with what you know. The first version I built of Hakken was using Python for both frontend and backend.

使用两者都有好处，但我认为坚持你所知道的会更好。我构建的 Hakken 的第一个版本在前端和后端都使用 Python。

For frontend I used Rich but soon enough I realized you cannot do much with Rich. Then I decided to build with Ink (React for terminal) so here you can do so much stuff. So I built every fancy thing that I could. But then it started flickering cause so many components and my poor frontend skills.

对于前端，我使用了 Rich，但很快我意识到用 Rich 做不了太多。然后我决定用 Ink（终端的 React）构建，所以在这里你可以做很多事情。所以我构建了我能构建的每一个花哨的东西。但后来它开始闪烁，因为有太多组件和我糟糕的前端技能。

Now I believe keeping it simple makes more sense. Cause we are working in terminal.

现在我相信保持简单更有意义。因为我们在终端中工作。

Hakken terminal agent interface 1

Hakken 终端代理界面 1

Figure 11. Hakken terminal agent in action — showcasing the clean CLI interface(kinda clean hehe).

图 11. 运行中的 Hakken 终端代理 — 展示干净的 CLI 界面（有点干净哈哈）。

### Some Important Decisions When Building Terminal Agents

### 构建终端代理时的一些重要决策

You can see that terminal-based AI agents have become standard for software development workflows(it great cause I moslty used terminal) and there are some critical things to get right:

你可以看到基于终端的 AI 代理已成为软件开发工作流程的标准（这很好，因为我主要使用终端），有一些关键的事情需要正确处理：

**TUI vs Simple CLI** - You gotta decide early if you want a full TUI (Terminal User Interface) or just a simple CLI. Most big tech companies like Anthropic and Google released terminal agents that had glitchy interfaces, but tools built with proper frameworks like Textual or BubbleTea are way smoother.

**TUI vs 简单 CLI** - 你必须早点决定是否想要完整的 TUI（终端用户界面）还是只是简单的 CLI。大多数大型科技公司如 Anthropic 和 Google 发布的终端代理都有故障界面，但用适当的框架如 Textual 或 BubbleTea 构建的工具要流畅得多。

**Output Display** - Think about what you want to show on terminal. Do you stream full output of tools? Do you show everything or just summaries? Modern terminal agents support features like Markdown rendering, syntax highlighting, and tables instead of plain text.

**输出显示** - 想想你想在终端上显示什么。你是否流式传输工具的完整输出？你是显示所有内容还是只显示摘要？现代终端代理支持 Markdown 渲染、语法高亮和表格等功能，而不是纯文本。

**Scrolling and Navigation** - This is huge. Users need to scroll through long outputs, search through responses, and navigate smoothly. Bad scrolling = bad experience.

**滚动和导航** - 这很重要。用户需要滚动浏览长输出，搜索响应，并平滑导航。糟糕的滚动 = 糟糕的体验。

## What Actually Matters

## 真正重要的是什么

After building Hakken from scratch, here's my hierarchy of what matters:

在从头开始构建 Hakken 之后，这是我认为重要的层次结构：

**1. Context Engineering** - Master this or everything else fails. Keep context tight and relevant. Use compression, tool result clearing, note-taking. Optimize KV-cache hit rates. Break work into short focused sessions under 200k tokens.

**1. 上下文工程** - 掌握这个，否则其他一切都会失败。保持上下文紧凑和相关。使用压缩、工具结果清除、笔记。优化 KV-cache 命中率。将工作分解为 200k 令牌以下的短时专注会话。

**2. Evaluation** - Build evals first, iterate with data. Start with unit tests for tools. Add model-based eval for scale. Use probe-based testing for context quality. The difference between a demo and a product is evaluation.

**2. 评估** - 首先构建评估，用数据迭代。从工具的单元测试开始。添加基于模型的评估以扩展规模。使用基于探针的测试来评估上下文质量。演示和产品之间的区别是评估。

**3. Own Your Stack** - Don't outsource to frameworks. Own your prompts - know every token that goes to the model. Own your control flow - handle edge cases when shit breaks. Own your context - decide what stays and goes.

**3. 掌控你的技术栈** - 不要外包给框架。掌控你的提示 - 知道发送给模型的每个令牌。掌控你的控制流 - 在出现问题时处理边缘情况。掌控你的上下文 - 决定什么保留什么删除。

**4. Simple Patterns** - Use what works. Subagents for complex tasks. Skills over MCPs for extensibility. Structured note-taking for memory. Progressive disclosure for context. Don't over-engineer.

**4. 简单模式** - 使用有效的方法。子代理用于复杂任务。Skills 而不是 MCPs 以实现可扩展性。结构化笔记用于记忆。渐进式披露用于上下文。不要过度工程化。

**5. Memory** - Only if you actually need it. Vertical apps (coding, finance) = useful. Horizontal apps (ChatGPT-style) = focus on context engineering instead.

**5. 记忆** - 仅在你真正需要时。垂直应用（编码、金融）= 有用。水平应用（ChatGPT 风格）= 专注于上下文工程。

## Wrap Up

## 总结

Look, building agents is still experimental science. These patterns work today but might change tomorrow. The field is moving fast. Stay flexible and iterate fast.

看，构建代理仍然是实验性科学。这些模式今天有效，但明天可能会改变。这个领域发展很快。保持灵活并快速迭代。

Thanks for reading this far! and Happy New Year !!

感谢你读到这里！新年快乐！！

Building Hakken was a great learning experience. And it's not still perfect but I learned a lot.

构建 Hakken 是一次很好的学习经验。它仍然不完美，但我学到了很多。

Find me on GitHub and Twitter, or buy me a coffee if this helped.

在 GitHub 和 Twitter 上找到我，如果这有帮助的话，请给我买杯咖啡。

Next I am working CUDA, LLM Inference, Fine Tuning and Reading more paper on long running agent and RL for context folding in 2026.

接下来我将致力于 CUDA、LLM 推理、微调，并在 2026 年阅读更多关于长时运行代理和用于上下文折叠的 RL 的论文。

Catch you later!

回头见！

---

*翻译完成 / Translation completed*
