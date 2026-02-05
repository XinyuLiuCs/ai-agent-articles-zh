# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository (https://github.com/XinyuLiuCs/ai-agent-articles-zh) contains Chinese translations of technical articles about AI agents, LLM applications, and AI research methodologies. Each document is a bilingual translation with Chinese content followed by the original English text.

**License**: MIT License (Copyright 2025 XinyuLiuCs)

## Repository Structure

The repository uses a flat structure with all 13 translation documents in the root directory, organized by topic:

- **AI研究方法论** (2 articles): bitter-lesson, learning-bitter-lesson
- **多智能体系统** (1 article): anthropic-multi-agent-research-system
- **上下文工程** (4 articles): effective-context-engineering, context-engineering-for-agents, context-engineering-for-ai-agents-lessons-from-building-manus, context-engineering-in-manus
- **智能体构建** (4 articles): building-effective-agents, equipping-agents-for-real-world, effective-harnesses-long-running-agents, hitchhikers-guide-llm-agent-complete
- **工具使用** (2 articles): introducing-advanced-tool-use, beyond-permission-prompts

## Document Structure

All markdown files follow a consistent format:
- Chinese title (H1)
- English title (secondary)
- Original article link
- Publication date
- Bilingual content: each English paragraph followed immediately by its Chinese translation
- Preserve all Markdown formatting, code blocks, headers, and structure

## File Naming Convention

Files use the pattern: `[original-title-slug]-zh.md` where `-zh` indicates Chinese translation.

## Repository Management

### Adding New Translations

When adding a new translation document:

1. **Create the translation file** following the naming convention
2. **Update README.md** with the new document:
   - Add entry under the appropriate category
   - Use format: `[**English Title 中文标题**](filename-zh.md) - Brief description`
   - Update the document count badge if needed
3. **Include original article link** at the top of the translation
4. **Stage and commit** with descriptive message

### Git Workflow

- **User identity**: `XinyuLiuCs <xinyu.liu.cs@gmail.com>`
- **Remote**: `git@github.com:XinyuLiuCs/ai-agent-articles-zh.git` (SSH)
- **Commit messages**: Use descriptive, bilingual commit messages with Co-Authored-By attribution
- When committing translation work, mention the article title and key improvements

### Acknowledgments

When adding new translations, check if new sources need to be added to the acknowledgments section in README.md. Current acknowledged sources include: ninehills, Anthropic, Lance Martin, Saurabh, and Manus team.

## Translation Workflow

### Core Translation Method: Two-Step Approach

For each English paragraph, use this internal process (do not expose intermediate steps in output):

1. **Literal Translation** - Translate sentence-by-sentence faithfully and completely, preserving all information
2. **Idiomatic Refinement** - Reorganize and polish based on Chinese reading habits for natural, fluent, professional expression

**Output only the final refined translation from Step 2.**

### Document Format

Each translated document must follow this structure:
- Chinese title (H1, primary)
- English title (plain text or italics, secondary)
- Publication date in both languages
- Bilingual content: each English paragraph followed immediately by its Chinese translation
- Preserve all Markdown formatting, code blocks, headers, and structure

### Translation Standards

**Language & Style:**
- Target: Simplified Chinese (简体中文)
- Tone: Professional, technical, objective, neutral
- Style: Engineering/research article conventions
- Do not add explanations or commentary not present in the original

**Technical Content:**
- Preserve technical terminology consistency across documents
- Keep code blocks, commands, and technical names unchanged
- Maintain exact heading hierarchy and semantic structure
- Do not translate: product names (Claude, Anthropic), technical terms in code

**Quality Checks:**
- Verify no information is lost or added
- Ensure natural Chinese expression without English sentence patterns
- Check consistency with existing translations in other documents
- Preserve all hyperlinks and references

### File Management

- Save completed translations in current working directory
- Use naming pattern: `[original-title-slug]-zh.md`
- Maintain UTF-8 encoding for Chinese characters
