# AI Agent 技术文章中文翻译集

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![文档数量](https://img.shields.io/badge/文档-19篇-blue.svg)]()
[![语言](https://img.shields.io/badge/语言-中文%2F英文-green.svg)]()

## 简介

本仓库收集了关于AI Agent、LLM和相关技术的高质量翻译文章，所有文档采用双语对照格式。

## 特点

-  **双语对照**：中英文对照，便于学习和理解
-  **技术准确**：保持专业术语一致性和技术内容准确性
-  **表达流畅**：符合中文阅读习惯的自然表达
-  **持续更新**：持续翻译和更新优质技术文章

## 文档列表

### AI研究方法论

- [**The Bitter Lesson 苦涩的教训**](bitter-lesson-zh.md) - Rich Sutton关于通用方法与算力的经典文章
- [**Learning the Bitter Lesson 学习苦涩的教训**](learning-bitter-lesson-zh.md) - 将Bitter Lesson应用于AI工程实践
- [**Measuring AI Ability to Complete Long Tasks 衡量AI完成长任务的能力**](measuring-ai-ability-complete-long-tasks-zh.md) - METR提出以任务长度衡量AI能力，发现该指标每7个月翻一番

### 多智能体系统

- [**How we built our multi-agent research system**](anthropic-multi-agent-research-system-zh.md) - Anthropic多智能体研究系统

### 上下文工程

- [**Effective context engineering for AI agents**](effective-context-engineering-for-ai-agents-zh.md) - Anthropic 上下文工程标志性文章，包括上下文压缩、SubAgent、Agentic Memory 等方法的介绍。
- [**Context Engineering for Agents**](context-engineering-for-agents-zh.md) - 上下文工程基础
- [**Context Engineering for AI Agents: Lessons from Building Manus**](context-engineering-for-ai-agents-lessons-from-building-manus-zh.md) - Peak总结Manus实践经验
- [**Context Engineering in Manus**](context-engineering-in-manus-zh.md) - Manus具体实践案例

### 智能体构建

- [**Building effective agents**](building-effective-agents-zh.md) - 将 Workflow 和 Autonomous Agent 拆分，并着重在未来 Agent 的发展。
- [**Equipping agents for the real world with Agent Skills**](equipping-agents-for-real-world-zh.md) - Agent Skills系统
- [**Effective harnesses for long-running agents**](effective-harnesses-long-running-agents-zh.md) - 长期运行智能体的有效框架
- [**ReAct: Synergizing Reasoning and Acting in Language Models 在语言模型中协同推理与行动**](react-synergizing-reasoning-acting-language-models-zh.md) - 提出推理与行动协同的经典范式，是当前AI Agent系统的重要理论基础
- [**The Hitchhikers Guide to LLM Agent**](hitchhikers-guide-llm-agent-complete-zh.md) - 从零构建编码智能体的完整经验

### 工具使用

- [**Introducing advanced tool use on the Claude Developer Platform**](introducing-advanced-tool-use-zh.md) - 介绍Tool Search Tool（工具搜索工具）、Programmatic Tool Calling（程序化工具调用）、Tool Use Examples（工具示例）三种范式
- [**Beyond permission prompts: making Claude Code more secure and autonomous**](beyond-permission-prompts-zh.md) - Claude Code Sandbox 机制的介绍
- [**Writing effective tools for agents — with agents 为智能体编写高效工具**](writing-effective-tools-for-agents-zh.md) - 编写高质量MCP工具与评估的完整方法论，含工具命名空间、上下文优化等原则
- [**Structured model outputs 结构化模型输出**](structured-model-outputs-zh.md) - OpenAI结构化输出功能详解，确保模型响应严格遵循JSON Schema定义

### API 与 SDK

- [**Building LLM-Powered Applications with Claude 使用Claude构建LLM驱动的应用**](claude-api-zh.md) - Claude API与Anthropic SDK完整开发指南，涵盖模型选择、工具调用、Agent SDK、流式传输、结构化输出等核心功能

### DevOps / CI/CD

- [**GitHub Actions**](github-actions-zh.md) - 使用GitHub Actions构建CI/CD流水线，涵盖工作流、触发器、缓存、矩阵构建、密钥管理、可复用工作流等完整实践指南

## 翻译标准

本项目遵循严格的翻译标准，详见 [CLAUDE.md](CLAUDE.md)。核心原则包括：

- **两步翻译法**：字面翻译 → 习惯表达优化
- **术语一致性**：保持技术术语在所有文档中的一致性
- **格式保留**：保留原文所有Markdown格式和结构
- **信息完整**：不添加或删除原文信息

## 如何使用

1. **在线浏览**：直接在GitHub上浏览文档
2. **clone到本地**：
   ```bash
   git clone https://github.com/XinyuLiuCs/ai-agent-articles-zh.git
   cd ai-agent-articles-zh
   ```
3. **独立阅读**：每篇文档都是独立的markdown文件，可单独阅读

## 许可证

本项目采用 [MIT License](LICENSE)。

### 版权声明

- 原文版权归原作者所有
- 翻译内容遵循MIT协议
- 使用时请注明原文出处和译者

## 致谢

感谢以下原作者和组织的优质内容：

- [**ninehilss**](https://github.com/ninehills/blog/issues/150) - 感谢大佬整理文章
- [**Anthropic**](https://www.anthropic.com/) - Claude相关技术文章
- [**Lance's Blog**](https://rlancemartin.github.io/) 
- [**Saurabh**](https://saurabhalone.com/)
- [**manus**](https://manus.im/blog)
- [**METR**](https://metr.org/) - AI能力评估与预测研究
- [**Shunyu Yao / Princeton NLP**](https://ysymyth.github.io/) - ReAct论文
- [**OpenAI**](https://openai.com/) - 结构化输出技术文档
- [**terminal-skills**](https://github.com/anthropics/claude-code) - GitHub Actions技能文档
- 其他贡献者
