# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Purpose

This repository contains Chinese translations of technical articles about Claude, AI agents, and AI research methodologies. Each document is a bilingual translation with Chinese content followed by the original English text.

## Document Structure

All markdown files follow a consistent format:
- Chinese title (primary)
- English title (secondary, in italics)
- Publication date
- Bilingual content throughout (Chinese paragraphs followed by English)

## Current Documents

1. **anthropic-multi-agent-research-system-zh.md** - Multi-agent research systems architecture and engineering lessons
2. **beyond-permission-prompts-zh.md** - Claude Code security features including sandboxing
3. **bitter-lesson-zh.md** - Rich Sutton's foundational essay on general methods and computation in AI research
4. **effective-context-engineering-for-ai-agents-zh.md** - Context engineering strategies for AI agents
5. **effective-harnesses-long-running-agents-zh.md** - Solutions for maintaining agent progress across multiple context windows
6. **equipping-agents-for-real-world-zh.md** - Agent Skills system for building specialized agents
7. **introducing-advanced-tool-use-zh.md** - Advanced tool use features on Claude Developer Platform
8. **learning-bitter-lesson-zh.md** - Applying the Bitter Lesson to AI engineering and application development

## File Naming Convention

Files use the pattern: `[original-title]-zh.md` where `-zh` indicates Chinese translation.

## Permissions

The `.claude/settings.local.json` file allows Claude Code to fetch content from www.anthropic.com for reference during translation work.

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
