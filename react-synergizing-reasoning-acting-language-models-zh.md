# ReAct：在语言模型中协同推理与行动

*ReAct: Synergizing Reasoning and Acting in Language Models*---
https://arxiv.org/abs/2210.03629

发布日期：2022年10月6日

Published Oct 6, 2022

## 摘要

Abstract

尽管大语言模型（LLM）在语言理解和交互式决策任务中展现出令人瞩目的能力，但它们在推理（如思维链提示）和行动（如动作计划生成）方面的能力在很大程度上被视为独立的研究课题。本文探索了如何让 LLM 以交错的方式同时生成推理轨迹和任务特定的行动，从而使两者之间产生更大的协同效应：推理轨迹帮助模型归纳、追踪和更新行动计划，以及处理异常情况；而行动则允许模型与外部信息源（如知识库或环境）进行交互，获取额外信息。我们将这种方法命名为 ReAct，并在多种语言和决策任务上进行了评估，展示了其相较于现有最优基线方法在性能上的优势，同时提升了人类的可解释性和可信度。具体而言，在问答（HotpotQA）和事实验证（Fever）任务上，ReAct 通过与简单的 Wikipedia API 交互，克服了思维链推理中普遍存在的幻觉和错误传播问题，生成了类似于人类的任务解决轨迹，比没有推理轨迹的基线方法更易于解释。在两个交互式决策基准（ALFWorld 和 WebShop）上，ReAct 分别以 34% 和 10% 的绝对成功率优势超越了模仿学习和强化学习方法，同时仅需一到两个上下文示例。

While large language models (LLMs) have demonstrated impressive capabilities across tasks in language understanding and interactive decision making, their abilities for reasoning (e.g. chain-of-thought prompting) and acting (e.g. action plan generation) have primarily been studied as separate topics. In this paper, we explore the use of LLMs to generate both reasoning traces and task-specific actions in an interleaved manner, allowing for greater synergy between the two: reasoning traces help the model induce, track, and update action plans as well as handle exceptions, while actions allow it to interface with external sources, such as knowledge bases or environments, to gather additional information. We apply our approach, named ReAct, to a diverse set of language and decision making tasks and demonstrate its effectiveness over state-of-the-art baselines, as well as improved human interpretability and trustworthiness. Concretely, on question answering (HotpotQA) and fact verification (Fever), ReAct overcomes issues of hallucination and error propagation prevalent in chain-of-thought reasoning by interacting with a simple Wikipedia API, and generates human-like task-solving trajectories that are more interpretable than baselines without reasoning traces. On two interactive decision making benchmarks (ALFWorld and WebShop), ReAct outperforms imitation and reinforcement learning methods by an absolute success rate of 34% and 10% respectively, while being prompted with only one or two in-context examples.

## 1 引言

1 Introduction

人类的一个独特之处在于能够将任务导向的行动与语言推理（或内心独白）无缝地结合起来，这被认为对人类的认知发展和自我调节具有重要意义（Vygotsky, 1987）。例如，想象你在厨房里准备做一道菜时的情景。在任意两个具体动作之间，我们可能会进行语言推理来追踪进度（"现在所有东西都切好了，我需要加热锅……"），处理异常或调整计划（"我没有盐了，就用酱油和胡椒代替吧"），以及认识到何时需要外部信息（"我要怎么准备面团？让我在网上搜一下"）。我们也可能采取行动（打开食谱书来查找食谱、打开烤箱）来支持推理并回答问题（"我应该把食材烤多久？……让我再看一遍食谱"）。这种"行动"与"推理"之间的紧密协同使人类能够快速学习新任务并执行稳健的决策或推理，即使在未曾预见的情况或面对信息不确定性时也是如此。

A unique feature of human intelligence is the ability to seamlessly combine task-oriented actions with verbal reasoning (or inner speech), which has been theorized to play an important role in human cognition for self-regulation or strategization (Vygotsky, 1987). Consider the example of cooking up a dish in the kitchen. Between any two specific actions, we may reason in language in order to track progress ("now that everything is cut, I should heat up the pan..."), to handle exceptions or adjust the plan according to the situation ("I don't have salt, so let me use soy sauce and pepper instead"), and to realize when external information is needed ("how do I prepare dough? Let me search on the Internet"). We may also act (open a cookbook to read the recipe, open the fridge, check ingredients) to support the reasoning and to answer questions ("What dish can I make right now?"). This tight synergy between "acting" and "reasoning" allows humans to learn new tasks quickly and perform robust decision making or reasoning, even under previously unseen circumstances or facing information uncertainties.

近期研究成果表明，将语言推理与交互式决策相结合的应用前景广阔，这一研究方向已经取得了大量自主体的成果。一方面，通过思维链（CoT）提示等技术，大语言模型在算术、常识和符号推理等需要多步推理的任务上展现出越来越强的能力（Wei et al., 2022）。然而，现有的这一方向的研究主要基于模型的内部表征来制定推理过程，未能借助外部世界信息进行建模，这限制了模型进行响应式推理以及根据现实情况更新知识的能力。

Recent results have hinted at the possibility of combining verbal reasoning with interactive decision making in autonomous systems. On one hand, properly prompted large language models (LLMs) have demonstrated emergent capabilities to carry out several steps of reasoning traces to derive answers from questions in arithmetic, commonsense, and symbolic reasoning tasks (Wei et al., 2022). However, this chain-of-thought (CoT) reasoning is a static black box, in that the model uses its own internal representations to generate thoughts and is not grounded in the external world, which limits its ability to reason reactively or update its knowledge.

另一方面，近期研究探索了使用大语言模型在交互式环境中进行行动和规划（如机器人操作、游戏玩法和网页导航），主要关注通过语言先验来预测动作。然而，这些方法并未利用语言模型进行抽象的、高层次的目标推理或维护工作记忆，这对于在复杂任务中有效行动至关重要。

On the other hand, recent work has explored the use of pre-trained language models for planning and acting in interactive environments (e.g. robotic manipulation, game playing, and web navigation), with a focus on predicting actions via language priors. However, they do not employ language models to reason abstractly about high-level goals or maintain a working memory to support acting, which is critical to performing complex tasks.

在本文中，我们提出了 ReAct——一种结合推理和行动的通用范式，使语言模型能够以交错的方式同时生成与任务相关的语言推理轨迹和行动，从而使模型能够执行动态推理以创建、维持和调整行动的高层计划（推理以指导行动），同时与外部环境（如 Wikipedia）进行交互，将额外信息纳入推理过程中（行动以辅助推理），如图 1 所示。

In this work, we present ReAct, a general paradigm to combine reasoning and acting with language models for solving diverse language reasoning and decision making tasks (Figure 1). ReAct prompts LLMs to generate both verbal reasoning traces and actions pertaining to a task in an interleaved manner, which allows the model to perform dynamic reasoning to create, maintain, and adjust high-level plans for acting (reason to act), while also interact with the external environments (e.g. Wikipedia) to incorporate additional information into reasoning (act to reason).

我们在四个不同的基准测试上对 ReAct 进行了评估：HotpotQA（Yang et al., 2018）和 FEVER（Thorne et al., 2018）用于知识密集型推理，ALFWorld（Shridhar et al., 2020b）和 WebShop（Yao et al., 2022）用于交互式决策。对于 HotpotQA 和 FEVER，通过少量上下文示例，ReAct 在使用 PaLM-540B（Chowdhery et al., 2022）进行提示时，相比仅进行行动（不含推理）的方法表现更优。同时，ReAct 在 FEVER 上的表现优于 CoT（Wei et al., 2022），在 HotpotQA 上与 CoT 的表现相当，但由于其基于与外部环境交互的推理，具有幻觉率显著更低的优势。我们还展示了 ReAct 和 CoT 之间的互补性，以及在模型规模较小时通过微调进行引导性学习与提示方法相比的优势。对于 ALFWorld 和 WebShop，仅使用一到两个上下文示例，ReAct 就分别以 34% 和 10% 的绝对成功率优势优于模仿学习/强化学习方法。

We perform experiments on four diverse benchmarks: HotpotQA (Yang et al., 2018) and FEVER (Thorne et al., 2018) for knowledge-intensive reasoning, and ALFWorld (Shridhar et al., 2020b) and WebShop (Yao et al., 2022) for interactive decision making. For HotpotQA and FEVER, with access to a Wikipedia API with search functionality, ReAct outperforms vanilla action generation models while being competitive with chain-of-thought reasoning (CoT) (Wei et al., 2022). The best approach overall is a combination of ReAct and CoT that allows for the use of both internal knowledge and externally obtained information during reasoning. On ALFWorld and WebShop, two or even one-shot ReAct prompting is able to outperform imitation or reinforcement learning methods trained with 10^3 ∼ 10^5 task instances, with an absolute improvement of 34% and 10% in success rates respectively.

## 2 ReAct：协同推理与行动

2 ReAct: Synergizing Reasoning + Acting

考虑一个通用的智能体在环境中与之交互以解决任务的设置。在时间步 $t$，智能体从环境中接收观测 $o_t \in \mathcal{O}$，并遵循某个策略 $\pi(a_t | c_t)$ 采取行动 $a_t \in \mathcal{A}$，其中 $c_t = (o_1, a_1, \ldots, o_{t-1}, a_{t-1}, o_t)$ 是智能体的上下文。将行动映射到自由形式的语言是自然的，因为语言模型可以利用语言来生成行动。

Consider a general setup of an agent interacting with an environment to solve a task. At time step $t$, an agent receives an observation $o_t \in \mathcal{O}$ from the environment and takes an action $a_t \in \mathcal{A}$ following some policy $\pi(a_t | c_t)$, where $c_t = (o_1, a_1, \ldots, o_{t-1}, a_{t-1}, o_t)$ is the context to the agent. Learning a policy is challenging when the mapping $c_t \mapsto a_t$ is highly implicit and requires extensive computation.

ReAct 的关键思想是将智能体的行动空间扩展为 $\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$，其中 $\mathcal{L}$ 是语言空间。语言空间中的行动 $\hat{a}_t \in \mathcal{L}$，我们称之为思考（thought）或推理轨迹，不影响外部环境，因此不会产生环境反馈。相反，思考 $\hat{a}_t$ 旨在通过对当前上下文 $c_t$ 进行推理来组合有用的信息，并更新上下文 $c_{t+1} = (c_t, \hat{a}_t)$ 以支持未来的推理或行动。

The key idea of ReAct is to augment the agent's action space to $\hat{\mathcal{A}} = \mathcal{A} \cup \mathcal{L}$, where $\mathcal{L}$ is the language space. An action $\hat{a}_t \in \mathcal{L}$ in the language space, which we will refer to as a thought or a reasoning trace, does not affect the external environment, thus leading to no observation feedback. Instead, a thought $\hat{a}_t$ aims to compose useful information by reasoning over the current context $c_t$, and update the context $c_{t+1} = (c_t, \hat{a}_t)$ to support future reasoning or acting.

如图 1 所示，思考可以有多种有用的类型，例如分解任务目标并创建行动计划，为行动注入与任务相关的常识知识，从观察中提取重要部分，追踪进度并转换行动计划，以及处理异常和调整行动计划等。

As shown in Figure 1, there could be various types of useful thoughts, e.g. decomposing task goals and create action plans, injecting commonsense knowledge relevant to task solving to act, extracting important parts from observations, track progress and transit action plans, and handle exceptions and adjust action plans, etc.

然而，由于语言空间 $\mathcal{L}$ 是无限的，学习这个增强空间中的策略是困难的且效果不保证。在本文中，我们主要关注使用冻结的大语言模型 PaLM-540B（Chowdhery et al., 2022）来通过少量上下文示例生成特定领域的行动和自由形式的语言思考，以实现 ReAct 范式（第 3 和第 4 节），同时也探索了在第 3.3 节中使用较小语言模型进行微调的方法。

However, since the language space $\mathcal{L}$ is unlimited, learning in this augmented action space is difficult and requires strong language priors. In this paper, we mainly focus on the setup where a frozen large language model, PaLM-540B (Chowdhery et al., 2022), is prompted with few-shot in-context examples to generate both domain-specific actions and free-form language thoughts for task solving (Section 3 and 4), and also explore fine-tuning smaller language models (Section 3.3).

对于生成推理轨迹和行动的具体方法，我们根据每个任务设计不同的提示，详见第 3 和第 4 节。一般而言，我们从训练集中选择一些样本，手动为其标注 ReAct 格式的轨迹（即交错的思考、行动、观察序列）作为少量示例。在以下部分中，我们设计了相应的思考类型，使得模型可以在行动和观察之间有效地进行推理，从而实现行动的计划、追踪和调整。

For the specific method of generating reasoning traces and actions, we design different prompts for each task, detailed in Sections 3 and 4. Broadly, we select a few cases from the training set, and manually compose ReAct-format trajectories (i.e., interleaved thought, action, observation sequences) as few-shot exemplars. In the following sections, we design proper thought types to enable models to reason and act effectively within interleaved action and observation steps.

> **图 1**：知识密集型推理任务（如 HotpotQA）和决策任务（如 ALFWorld）中四种提示方法的比较。在这两种类型的任务中，ReAct 不仅能够产生行动（Act），还能生成任务相关的推理轨迹（Thought），使得推理和行动之间形成更强的协同效应。

> **Figure 1**: Comparison of four prompting methods, (a) Standard, (b) Chain-of-thought (CoT, Reason Only), (c) Act-only, and (d) ReAct (Reason+Act), solving a HotpotQA (Yang et al., 2018) and an AlfWorld (Shridhar et al., 2020b) question. In both domains, we omit input prompts (i.e., in-context examples) for simplicity. We note that tasks may require many more actions than what is shown here to solve.

## 3 知识密集型推理

3 Knowledge-Intensive Reasoning Tasks

### 3.1 实验设置

3.1 Setup

我们考虑两个需要在多步骤中进行推理的知识密集型任务基准：(1) HotpotQA（Yang et al., 2018），一个多跳问答基准，需要对两个或更多 Wikipedia 段落进行推理；(2) FEVER（Thorne et al., 2018），一个事实验证基准，每个声明被标注为 SUPPORTS（支持）、REFUTES（反驳）或 NOT ENOUGH INFO（信息不足）。在这两个基准中，我们仅使用问题或声明作为输入来操作模型，不提供支持段落，这使得检索外部知识对于成功完成任务成为必要。

We consider two knowledge-intensive reasoning task benchmarks requiring multi-step reasoning: (1) HotpotQA (Yang et al., 2018), a multi-hop question answering benchmark that requires reasoning over two or more Wikipedia passages, and (2) FEVER (Thorne et al., 2018), a fact verification benchmark where each claim is annotated SUPPORTS, REFUTES, or NOT ENOUGH INFO. In this work, we operate in a question-only setup for both tasks, where models only receive the question/claim as input without access to support paragraphs, and have to rely on their internal knowledge or retrieve knowledge via interacting with an external environment to support reasoning.

我们设计了一个简单的 Wikipedia Web API，包含三种类型的动作来支持交互式信息检索：(1) **search[entity]**，返回对应 Wikipedia 实体页面的前 5 个句子（如果存在），否则返回 Wikipedia 搜索引擎中的前 5 个相似实体；(2) **lookup[string]**，返回页面中包含该字符串的下一个句子，模拟浏览器中的 Ctrl+F 功能；(3) **finish[answer]**，用给定的答案完成当前任务。我们注意到，这个动作空间大多只能检索信息片段，比完整的 Wikipedia 段落粒度更细，并且不支持类似于常用信息检索方法中的排序功能。

We design a simple Wikipedia web API with three types of actions to support interactive information retrieval: (1) **search[entity]**, which returns the first 5 sentences from the corresponding entity wiki page if it exists, or else suggests top-5 similar entities from the Wikipedia search engine, (2) **lookup[string]**, which would return the next sentence in the page containing string, simulating Ctrl+F functionality on the browser, and (3) **finish[answer]**, which would finish the current task with answer. We note that this action space mostly can only retrieve a small part of a passage based on exact entity name or string, which is significantly weaker than state-of-the-art lexical or neural retrievers.

### 3.2 方法

3.2 Methods

**ReAct 提示**：对于 HotpotQA 和 FEVER，我们分别从训练集中随机选择 6 个和 3 个样本，手动编写 ReAct 格式的轨迹，用作少量提示中的示例。与图 1(d) 类似，每条轨迹由多个思考-行动-观察步骤组成，其中自由形式的思考用于多种目的。具体来说，我们使用思考的组合来：分解问题（"我需要搜索 x，找到 y，然后找到 z"）；从 Wikipedia 观察中提取信息（"x 于 1844 年启动"、"该段落没有告诉我 x"）；执行常识（"x 不是 y，所以 z 一定是……"）或算术推理（"1844 < 1989"）；指导搜索的重新表述（"也许我应该改为搜索 x"）；以及综合最终答案（"……所以答案是 x"）。

**ReAct Prompting**: For HotpotQA and FEVER, we randomly select 6 and 3 cases respectively from the training set and manually compose ReAct-format trajectories to use as few-shot exemplars in the prompts. Similar to Figure 1(d), each trajectory consists of multiple thought-action-observation steps (with free-form thoughts used for various purposes). Specifically, we use a combination of thoughts that decompose questions ("I need to search x, find y, then find z"), extract information from Wikipedia observations ("x was started in 1844", "The paragraph does not tell x"), perform commonsense ("x is not y, so z must be...") or arithmetic reasoning ("1844 < 1989"), guide search reformulation ("maybe I should search x instead"), and synthesize the final answer ("...so the answer is x").

**基线方法**：我们系统地对以下基线方法进行了消融比较：(a) Standard 提示（移除所有思考、行动和观察）；(b) Chain-of-thought 提示（CoT）（Wei et al., 2022）（移除行动和观察）；(c) 带自一致性的 CoT（CoT-SC）（Wang et al., 2022b）（使用温度 0.7 采样 21 条 CoT 轨迹，取多数投票结果）；(d) 仅行动提示（Act）（移除思考）。

**Baselines**: We systematically ablate ReAct trajectories to build prompts for the following baselines: (a) Standard prompting (Standard), which removes all thoughts, actions and observations in ReAct trajectories. (b) Chain-of-thought prompting (CoT) (Wei et al., 2022), which removes actions and observations and serves as a reasoning-only baseline. (c) CoT with self-consistency (CoT-SC) (Wang et al., 2022b), which samples 21 CoT trajectories with decoding temperature 0.7 and takes the majority answer, serving as the strongest reasoning-only baseline. (d) Acting-only prompt (Act), which removes thoughts in ReAct trajectories, serving as a baseline without reasoning.

**ReAct 与 CoT-SC 的组合方法**：我们还提出了两种结合 ReAct 和 CoT-SC 的策略，并根据以下启发式方法在二者之间切换：(a) ReAct → CoT-SC：当 ReAct 在给定步数内未能返回答案时，切换到 CoT-SC；(b) CoT-SC → ReAct：当 $n$ 次 CoT-SC 采样中多数答案的出现次数低于 $n/2$ 时（即内部知识不足以自信地支持该任务），切换到 ReAct。

**Combining ReAct and CoT-SC**: We also propose strategies to combine ReAct with CoT-SC, and switch between the two based on the following heuristics: (A) ReAct → CoT-SC: when ReAct fails to return an answer within given steps, back off to CoT-SC. (B) CoT-SC → ReAct: when the majority answer among $n$ CoT-SC samples occurs less than $n/2$ times (i.e. the internal knowledge does not confidently support the task), back off to ReAct.

**微调**：鉴于少量提示方法的局限性，我们使用 3,000 条由 ReAct 格式生成的、答案正确的轨迹来微调较小的语言模型（PaLM-8B 和 PaLM-62B），以在更经济的模型规模上验证方法的有效性。

**Finetuning**: Due to the limited context window of LLMs for few-shot prompting, we also explore finetuning smaller language models (PaLM-8/62B) with 3,000 ReAct-format trajectories whose final answers are correct, to test the effectiveness of ReAct at more affordable model scales.

### 3.3 结果与观察

3.3 Results and Observations

**ReAct 与基线方法比较（表 1）**：表 1 展示了 HotpotQA 和 FEVER 上不同提示方法的结果。在 PaLM-540B 上，ReAct 在 Fever 上优于 Act（60.9 vs. 58.9），在 HotpotQA 上也优于 Act（27.4 vs. 25.7），这证明了为任务解决生成推理轨迹的价值。ReAct 在 Fever 上的表现也优于 CoT（60.9 vs. 56.3），但在 HotpotQA 上表现不如 CoT（27.4 vs. 29.4），可能是因为一些包含多步信息检索的 ReAct 轨迹更具挑战性。同时，ReAct 在两个任务上均不如 CoT-SC（其使用 21 个采样），但当与 CoT-SC 组合时，两种策略均能进一步提升性能：ReAct → CoT-SC 在 HotpotQA 上达到了 35.1 的最优性能，CoT-SC → ReAct 在 FEVER 上达到了 64.6 的最优性能。

**ReAct vs. Baselines (Table 1)**: Table 1 shows HotpotQA and FEVER results using different prompting methods with PaLM-540B as the base model. We note that ReAct is better than Act on both tasks, demonstrating the value of reasoning to guide acting, especially for synthesizing the final answer. On the other hand, ReAct outperforms CoT on Fever (60.9 vs. 56.3) and is on par with CoT on HotpotQA (27.4 vs. 29.4), with the advantage of being grounded and interpretable. The best prompting method on HotpotQA is ReAct → CoT-SC (35.1) and the best on FEVER is CoT-SC → ReAct (64.6), both combining the strengths of internal and external knowledge.

| 提示方法 Prompting Method | HotpotQA (EM) | FEVER (Acc) |
|---|---|---|
| Standard | 28.7 | 57.1 |
| CoT (Wei et al., 2022) | 29.4 | 56.3 |
| CoT-SC (Wang et al., 2022b) | 33.4 | 60.4 |
| Act | 25.7 | 58.9 |
| ReAct | 27.4 | 60.9 |
| CoT-SC → ReAct | 35.1 | 62.0 |
| ReAct → CoT-SC | 34.2 | 64.6 |

> **表 1**：PaLM-540B 在 HotpotQA 和 FEVER 上使用不同提示方法的结果（500 个随机样本上的表现）。HotpotQA 使用精确匹配（EM）评估，FEVER 使用准确率（Acc）评估。最优结果用粗体标出。ReAct 优于 Act，组合方法在两个任务上均取得了最优性能。

> **Table 1**: PaLM-540B prompting results on HotpotQA and FEVER (500 random samples). EM = exact match. Best results in bold. ReAct outperforms Act, and combining methods outperforms all individual approaches on both tasks.

**ReAct vs. CoT（内部知识 vs. 外部知识）**：CoT 和 ReAct 之间存在有趣的互补关系，分别依赖于内部知识和外部知识。我们随机抽取 50 条正确和 50 条错误的 ReAct 与 CoT 轨迹（HotpotQA）进行了人工分析。

**ReAct vs. CoT (Internal vs. External Knowledge)**: An interesting finding is the complementary nature of CoT and ReAct, which rely on internal and external knowledge respectively. We randomly sampled 50 correct and 50 incorrect trajectories from ReAct and CoT (for HotpotQA) for manual analysis.

在成功案例中，ReAct 的正确率为 94%（正确推理和事实），仅有 6% 的幻觉率，而 CoT 的正确率为 86%，幻觉率为 14%。这表明，得益于对外部知识库的访问，ReAct 的问题解决轨迹更加扎实、基于事实且值得信赖。

Among success cases, ReAct true positive rate was 94% (correct reasoning and facts) versus 86% for CoT. ReAct hallucination rate was only 6% compared to 14% for CoT. This demonstrates that ReAct's problem solving trajectory is more grounded, fact-driven, and trustworthy, thanks to the access of an external knowledge base.

在失败案例中，ReAct 的主要失败模式是推理错误（47%）和搜索结果不成功（23%），而 CoT 的主要失败模式是幻觉（56%）。具体而言，CoT 的大部分错误来自于幻觉事实（56%），而 ReAct 的幻觉率为 0%。ReAct 的推理错误率（47%）高于 CoT（16%），这在一定程度上是因为 ReAct 的搜索结果有时会分散模型的推理注意力。

Among failure cases, ReAct's main failure modes were reasoning errors (47%) and unsuccessful search results (23%), while CoT's dominant failure mode was hallucination (56%). Specifically, CoT hallucinated facts in 56% of error cases, while ReAct had 0% hallucination. ReAct's reasoning error rate (47%) was higher than CoT's (16%), partly because search results sometimes distracted the model's reasoning.

这些互补特征解释了为什么组合方法效果最佳：当 ReAct 在信息检索上遇到困难时，CoT 的内部知识可以提供帮助；当 CoT 遭受幻觉时，ReAct 可以通过外部信息进行纠正。

These complementary characteristics explain why the combined approaches work best: when ReAct struggles with information retrieval, CoT's internal knowledge can help; when CoT suffers from hallucination, ReAct can correct it with external information.

> **图 2**：PaLM-540B 在不同数量的 CoT-SC 样本下的性能。ReAct + CoT-SC 的组合方法仅需 3-5 个样本就能达到 21 个样本的 CoT-SC 的性能水平。

> **Figure 2**: PaLM-540B prompting scaling curves with respect to the number of CoT-SC samples used on HotpotQA and FEVER. ReAct + CoT-SC performance reaches CoT-SC levels using merely 3-5 samples versus 21 samples.

**微调结果（图 3）**：我们使用 3,000 条答案正确的轨迹在 PaLM-8B 和 PaLM-62B 上微调了四种方法（Standard、CoT、Act、ReAct）。在四种方法中，ReAct 是微调效果最好的方法。在 HotpotQA 上，ReAct 微调后的 PaLM-8B 已经超过了所有 PaLM-62B 的提示方法，而 ReAct 微调后的 PaLM-62B 则超过了所有 PaLM-540B 的提示方法。

**Finetuning Results (Figure 3)**: We finetune four methods (Standard, CoT, Act, ReAct) using 3,000 trajectories with correct answers on PaLM-8B and PaLM-62B. Among the four methods, ReAct is the best finetuning format. On HotpotQA, ReAct finetuned PaLM-8B already outperforms all PaLM-62B prompting methods, and ReAct finetuned PaLM-62B outperforms all PaLM-540B prompting methods.

> **图 3**：HotpotQA 上不同方法在提示和微调设置下的模型规模扩展结果。微调使用 3,000 条正确轨迹。ReAct 是所有方法中微调效果最好的格式。

> **Figure 3**: Scaling results for prompting and fine-tuning on HotpotQA across Standard, CoT, Act, and ReAct methods with different model sizes (PaLM-8/62/540B). Finetuning uses 3,000 correct trajectories. ReAct is the best finetuning format.

> **图 4**：一个 HotpotQA 示例，原始数据集标签已过时；ReAct 能够通过与 Wikipedia 交互检索到当前最新的信息，从而给出正确答案。

> **Figure 4**: An example HotpotQA question where original dataset labels were outdated; ReAct retrieved current information through web interaction and provided the correct answer.

## 4 决策任务

4 Decision Making Tasks

我们还在两个基于语言的交互式决策任务上测试了 ReAct：ALFWorld 和 WebShop，这两个任务都要求智能体在复杂的环境中执行长序列的行动。

We also test ReAct on two language-based interactive decision making tasks: ALFWorld and WebShop, both of which feature complex environments that require agents to act over long horizons with sparse rewards.

### ALFWorld

ALFWorld（Shridhar et al., 2020b）是一个与具身 ALFRED 基准（Shridhar et al., 2020a）对齐的合成文本游戏环境，包含 6 种类型的任务，智能体需要通过文本动作在模拟的家居环境中导航并与物品交互以完成高层目标（例如"在台灯下检查报纸"）。一个任务实例可能涉及超过 50 个位置，专家策略可能需要超过 50 步才能解决。这要求智能体规划子目标、进行系统性探索，并利用常识知识来推断物品可能的位置。

ALFWorld (Shridhar et al., 2020b) is a synthetic text-based game aligned with the embodied ALFRED benchmark (Shridhar et al., 2020a), including 6 types of tasks in which the agent needs to achieve a high-level goal (e.g. examine paper under desklamp) by navigating and interacting with a simulated household via text actions (e.g. go to coffeetable 1, take paper 2, use desklamp 1). A task instance can involve more than 50 locations and expert policy may require more than 50 steps to solve. This task is challenging as it requires the agent to plan sub-goals, perform systematic exploration, and leverage commonsense knowledge to reason about item locations.

我们为每种任务类型标注了 3 条 ReAct 格式的训练轨迹。与知识密集型任务中的密集思考-行动-观察交替不同，决策任务中的推理轨迹较为稀疏——我们仅在智能体需要分解目标、追踪子目标完成情况、确定下一步行动或应用常识推理时插入思考。例如，"我需要找到并拿起一个苹果，然后把它清洗干净。苹果更可能出现在冰箱(1)、台面(1)、桌子(1)处。让我先去冰箱(1)看看。"

We annotate 3 ReAct-format training trajectories for each task type. Unlike the dense thought-action-observation alternation in knowledge reasoning, decision making tasks feature sparse reasoning traces — we only insert thoughts when the agent needs to decompose goals, track sub-goal completion, determine the next action, or apply commonsense reasoning. For example, "I need to find and take an apple, then wash it. Apples are more likely to appear in fridge (1), countertop (1), table (1). Let me go to fridge (1) first."

对于基线方法 Act，我们使用相同的轨迹但移除了思考。为了测试提示的鲁棒性，我们通过不同的轨迹排列创建了 6 种受控的提示变体，并报告了平均和最优结果。我们还与 BUTLER（Shridhar et al., 2020b）进行了比较，这是一种基于模仿学习训练的方法。

For the Act baseline, we use the same trajectories with thoughts removed. To test prompting robustness, we create 6 controlled prompt variations through different trajectory permutations and report both average and best results. We also compare with BUTLER (Shridhar et al., 2020b), an imitation learning method.

| 方法 Method | Pick | Clean | Heat | Cool | Look | Pick 2 | All |
|---|---|---|---|---|---|---|---|
| BUTLER (Shridhar et al., 2020b) | 46 | 39 | 74 | 100 | 22 | 24 | 37 |
| Act (best of 6) | 88 | 42 | 74 | 67 | 72 | 41 | 45 |
| ReAct (avg of 6) | 65 | 39 | 83 | 76 | 55 | 24 | 57 |
| ReAct (best of 6) | 92 | 58 | 96 | 86 | 78 | 41 | 71 |

> **表 3**：ALFWorld 上不同任务类型的成功率（%）。ReAct 在平均和最优结果上均显著优于 Act 和 BUTLER 基线。

> **Table 3**: ALFWorld task-specific success rates (%). ReAct significantly outperforms both Act and BUTLER baselines in average and best results.

ReAct 在所有 6 种提示变体中平均成功率达到 57%，最优达到 71%，显著优于 Act 的最优 45% 和 BUTLER 的 37%。值得注意的是，即使是 ReAct 的最差试验（48%）也超过了两种对比方法。

ReAct achieves an average success rate of 57% and best of 71% across 6 prompt variations, significantly outperforming Act's best of 45% and BUTLER's 37%. Notably, even ReAct's worst trial (48%) exceeds both comparison methods.

**内部推理 vs. 外部反馈分析**：我们引入了 ReAct-IM 消融实验，将思考替换为仅对环境反馈做出简单反应的风格（类似于 Inner Monologue 方法），以比较内部推理和外部反馈的作用。结果显示，ReAct 显著优于 ReAct-IM（71% vs. 53% 的整体成功率），在六种任务中的五种上具有一致的优势。这说明 ReAct 在高层目标分解和常识推理方面的能力是 ReAct-IM 所缺乏的。

**Internal Reasoning vs. External Feedback Analysis**: We introduced a ReAct-IM ablation replacing thoughts with simple reaction-style observations (similar to the Inner Monologue approach) to compare internal reasoning and external feedback. Results showed ReAct substantially outperformed ReAct-IM (71% vs. 53% overall success rate), with consistent advantages on five of six tasks. This demonstrates that ReAct's capacity for high-level goal decomposition and commonsense reasoning is lacking in IM-style approaches.

### WebShop

WebShop（Yao et al., 2022）是一个在线购物网站环境，包含 118 万个真实商品和 12,000 条人工指令。智能体需要根据用户指令通过网页交互购买匹配特定条件的商品。与 ALFWorld 不同的是，WebShop 包含大量结构化和非结构化文本（商品标题、描述、来自 Amazon 的选项信息），需要智能体在海量信息中有效浏览和决策。评估使用 500 条测试指令，度量指标包括平均得分（属性覆盖百分比）和成功率。

WebShop (Yao et al., 2022) is an online shopping website environment with 1.18M real-world products and 12,000 human instructions. The agent needs to purchase products matching user specifications through web interactions. Unlike ALFWorld, WebShop contains massive amounts of structured and unstructured text (product titles, descriptions, Amazon-sourced options), requiring agents to effectively browse and make decisions among vast information. Evaluation uses 500 test instructions with average score (attribute coverage percentage) and success rate as metrics.

我们使用 1-2 个上下文示例进行 ReAct 提示，其中包含关于探索策略、购买时机和产品相关性的推理思考。

We use 1-2 in-context examples for ReAct prompting, which include reasoning thoughts about exploration strategy, purchase timing, and product relevance.

| 方法 Method | Score | SR (%) |
|---|---|---|
| IL (Yao et al., 2022) | 59.9 | 29.1 |
| IL + RL (Yao et al., 2022) | 62.4 | 28.7 |
| Act | 62.3 | 30.1 |
| ReAct | 66.6 | 40.0 |
| Human Expert | 82.1 | 59.6 |

> **表 4**：WebShop 上的结果。Score 为平均得分，SR 为成功率。ReAct 以绝对 10% 的成功率优势超越了模仿学习（IL）和强化学习（IL+RL）方法，同时也显著优于仅行动（Act）基线。

> **Table 4**: WebShop results. Score = average score. SR = success rate. ReAct outperforms imitation learning (IL) and reinforcement learning (IL+RL) methods by an absolute 10% in success rate, and also significantly outperforms the Act-only baseline.

ReAct 取得了 40.0% 的成功率和 66.6 的平均得分，相比 Act 的 30.1% 成功率和 62.3 的得分有显著提升，同时也超越了基于 10^3 ∼ 10^5 条任务实例训练的模仿学习（29.1%）和 IL+RL（28.7%）方法。

ReAct achieves a 40.0% success rate and 66.6 average score, a significant improvement over Act's 30.1% success rate and 62.3 score, while also surpassing imitation learning (29.1%) and IL+RL (28.7%) methods trained with 10^3 ∼ 10^5 task instances.

**人类参与实验**：我们还展示了 ReAct 在人机协作方面的优势。通过编辑少量推理轨迹（如移除幻觉内容、添加有针对性的提示），可以比修改多个具体行动更有效地重新引导智能体的行为。这种通过修改思考来纠正行为的方式为人机协作提供了新的可能性，这是纯行动方法或强化学习方法所不具备的。

**Human-in-the-Loop**: We also demonstrate ReAct's advantage in human-machine collaboration. By editing a few reasoning traces (such as removing hallucinated content and adding targeted hints), agent behavior can be redirected more effectively than modifying multiple specific actions. This way of correcting behavior through thought editing opens new possibilities for human-machine collaboration unavailable to action-only or RL methods.

## 5 相关工作

5 Related Work

**语言模型推理**：也许最相关的先驱工作是思维链（CoT）提示（Wei et al., 2022），它揭示了 LLM 能够为问题解决制定自己的"思考过程"的能力。后续发展包括最少到最多提示（least-to-most prompting）（Zhou et al., 2022）、零样本 CoT（zero-shot-CoT）（Kojima et al., 2022）和自一致性（self-consistency）方法（Wang et al., 2022b）等。近期的工作还研究了 CoT 的结构和符号模式。另一类方法将推理过程分解为由专门的语言模型执行的离散步骤。与这些方法不同，ReAct 不仅仅在内部进行推理，还通过与外部环境的交互来增强推理过程。

**Language Model Reasoning**: Perhaps the most relevant prior work is chain-of-thought (CoT) prompting (Wei et al., 2022), which reveals the ability of LLMs to formulate their own "thinking procedure" for problem solving. Subsequent developments include least-to-most prompting (Zhou et al., 2022), zero-shot-CoT (Kojima et al., 2022), and self-consistency methods (Wang et al., 2022b). Recent work has also examined CoT structure and symbolic patterns. Alternative architectures decompose reasoning into discrete steps performed by dedicated language models. Unlike these methods, ReAct augments reasoning not just internally, but through interaction with external environments.

**语言模型决策**：先前的工作将 LLM 应用于交互式任务，包括网页浏览智能体和面向任务的对话系统。WebGPT（Nakano et al., 2021）使用语言模型与网页浏览器交互，但依赖于昂贵的人类反馈进行强化学习。ReAct 与这些方法的区别在于仅使用语言描述，无需昂贵的策略学习数据集。SayCan（Ahn et al., 2022）和 Inner Monologue（Huang et al., 2022b）是具身 AI 领域最相关的前驱工作。然而，ReAct 通过灵活且稀疏的推理轨迹区别于它们，而非依赖密集的环境反馈。此外，越来越多的研究将语言作为语义丰富的输入用于各种交互式环境中的决策。

**Decision Making with Language Models**: Prior work applies LLMs to interactive tasks, including web browsing agents and task-oriented dialogue systems. WebGPT (Nakano et al., 2021) uses language models to interact with web browsers but relies on expensive human feedback for reinforcement learning. ReAct contrasts with these by using only language descriptions without requiring costly policy learning datasets. SayCan (Ahn et al., 2022) and Inner Monologue (Huang et al., 2022b) represent closest prior work in embodied AI. However, ReAct distinguishes itself through flexible and sparse reasoning traces rather than dense environmental feedback. Additionally, research increasingly uses language as semantically-rich input for interactive decision-making across diverse environments.

## 6 结论

6 Conclusion

本文提出了 ReAct，一种在大语言模型中协同推理和行动的简单而有效的方法。通过在问答、事实验证和交互式决策等多种任务上的实验，我们展示了 ReAct 在性能上的优越性以及可解释性和可信度方面的提升。ReAct 在知识密集型任务上的成功案例分析表明，它的问题解决过程更加扎实、基于事实且可信赖。在决策任务上，ReAct 仅需一到两个示例就能超越使用数千条训练数据的模仿学习和强化学习方法。

In this paper, we have presented ReAct, a simple yet effective method for synergizing reasoning and acting in large language models. Through experiments on question answering, fact verification, and interactive decision making tasks, we demonstrate ReAct's superiority in performance as well as improved interpretability and trustworthiness. Success case analysis on knowledge-intensive tasks shows ReAct's problem solving process is more grounded, fact-driven, and trustworthy. On decision making tasks, ReAct outperforms imitation and reinforcement learning methods trained with thousands of examples using only one or two demonstrations.

我们也承认当前方法存在的局限性。对于知识密集型推理任务，尽管通过与外部环境交互可以在一定程度上弥补内部知识的不足，但复杂任务中的大规模动作空间可能需要更多的示范样本，这可能超出上下文学习的输入长度限制。我们在微调方面的初步探索展示了令人鼓舞的结果，但从更多高质量人工标注中学习将是进一步提升性能的关键。未来的研究方向包括通过多任务训练扩展 ReAct 的规模，以及将其与强化学习等互补范式相结合。

We also acknowledge limitations of the current approach. For knowledge-intensive reasoning tasks, while interacting with external environments can partially compensate for internal knowledge deficiencies, complex tasks with large action spaces may require more demonstrations that could exceed in-context learning input limits. Our initial explorations with finetuning show promising results, but learning from more high-quality human annotations will be the desiderata to further improve performance. Future directions include scaling up ReAct with multi-task training and combining it with complementary paradigms like reinforcement learning.
