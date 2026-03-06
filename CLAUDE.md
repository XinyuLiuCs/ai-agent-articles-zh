# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository (https://github.com/XinyuLiuCs/ai-agent-articles-zh) contains Chinese translations of technical articles about AI agents, LLM applications, and AI research methodologies.

**License**: MIT License (Copyright 2025 XinyuLiuCs)

## Repository Structure

Flat structure with all translation documents in the root directory. Files use the naming pattern `[original-title-slug]-zh.md`.

Categories (17 articles total):

- **AI研究方法论** (3 articles): bitter-lesson, learning-bitter-lesson, measuring-ai-ability-complete-long-tasks
- **多智能体系统** (1 article): anthropic-multi-agent-research-system
- **上下文工程** (4 articles): effective-context-engineering-for-ai-agents, context-engineering-for-agents, context-engineering-for-ai-agents-lessons-from-building-manus, context-engineering-in-manus
- **智能体构建** (5 articles): building-effective-agents, equipping-agents-for-real-world, effective-harnesses-long-running-agents, hitchhikers-guide-llm-agent-complete, react-synergizing-reasoning-acting-language-models
- **工具使用** (4 articles): introducing-advanced-tool-use, beyond-permission-prompts, writing-effective-tools-for-agents, structured-model-outputs

## Document Format

Each translated document follows this structure:

```
# 中文标题

*English Title*---
https://original-article-url

发布日期：YYYY年M月D日

Published Mon DD, YYYY

中文段落翻译

English original paragraph

中文段落翻译

English original paragraph
```

Key rules:
- **Bilingual ordering**: Chinese paragraph first, then the corresponding English original paragraph
- English title line uses italics with `*` and ends with `---` (some older files vary slightly)
- Preserve all Markdown formatting, code blocks, headers, and structure
- Headings are bilingual: Chinese H2/H3 first, then English on next line

## Adding New Translations

1. **Create the file** following the naming convention `[original-title-slug]-zh.md`
2. **Update README.md**:
   - Add entry under the appropriate category using format: `[**English Title 中文标题**](filename-zh.md) - Brief description`
   - Update the document count badge number
   - Add new sources to the acknowledgments section if needed
3. **Include original article link** at the top of the translation

## Git Workflow

- **User identity**: `XinyuLiuCs <xinyu.liu.cs@gmail.com>`
- **Remote**: `git@github.com:XinyuLiuCs/ai-agent-articles-zh.git` (SSH)
- **Commit messages**: Descriptive, mention article title and key changes

## Translation Method

Use this two-step internal process (output only the final result):

1. **Literal Translation** — Translate sentence-by-sentence faithfully, preserving all information
2. **Idiomatic Refinement** — Reorganize and polish for natural, professional Chinese expression

### Standards

- Target: Simplified Chinese (简体中文), professional/technical tone
- Do not add explanations or commentary not in the original
- Do not translate: product names (Claude, Anthropic, MCP), technical terms in code blocks
- Preserve technical terminology consistency across all documents in the repository
- Preserve all hyperlinks and references
- Maintain UTF-8 encoding

### Acknowledged Sources

Current sources in README.md: ninehills, Anthropic, Lance Martin, Saurabh, Manus team, METR, OpenAI.
