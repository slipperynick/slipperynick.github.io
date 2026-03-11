---
title: "Semgrep - AI Security Best Practices Ruleset"
date: 2026-03-10 09:30:00 +0100
categories: [security, ai, appsec]
tags: [llm, semgrep, mcp, prompt-injection, code-review]
---

## Context

Recently I have been reviewing a few AI related applications at work that cover LLMs, MCP, Langchain and other adjacent technology. It was fortuitous timing that Semgrep released a free ruleset that aims to encourage AI best practices by finding the bad practices.

The categories include

- Hardcoded API keys
- Prompt injection
- Missing refusal handling
- Missing safety settings
- No error handling
- Missing moderation
- Hooks security
- MCP server flaws
- Agentic code execution
- Config file attacks


## Quick Test

Installation was covered by brew and I leveraged Claude Code to build a sample application for testing purposes. I have been pretty impressed with Claude Code, which has also build out a whole risk assessment process for me in a few prompts.

![semgrep1](/assets/img/posts/2026-03-09-semgrep-ai-bestpractices/semgrep_ai_bestpractices.gif)


I plan to do a bit more research into each of categories and how detection is done in the Semgrep rules. 

![semgrep1](/assets/img/posts/2026-03-09-semgrep-ai-bestpractices/semgrep_ai_bestpractices2.png)


# References
- https://semgrep.dev
- https://github.com/semgrep/ai-best-practices