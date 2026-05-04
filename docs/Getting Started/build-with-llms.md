---
title: Build with LLMs
excerpt: Use LLMs in your SSTrader integration workflow.
deprecated: false
hidden: false
metadata:
  robots: index
---
You can leverage Large Language Models (LLMs) to streamline the development of SSTrader integrations. We offer a comprehensive suite of tools designed to provide your AI assistant with precise, real-time context regarding the SSTrader API.

| Tool           | What it does                                                            | Best for                                                              |
| -------------- | ----------------------------------------------------------------------- | --------------------------------------------------------------------- |
| **MCP server** | Lets your assistant search and read these docs live during your session | IDE-based workflows (Claude Code, Cursor, VS Code)                    |
| **Skill**      | Teaches your assistant how to approach SSTrader integration tasks       | Pairing with MCP for deeper guidance on workflows and common mistakes |
| **llms.txt**   | Plain-text index of the documentation                                   | Pasting context into ChatGPT, Gemini, or claude.ai conversations      |

## MCP
**Model Context Protocol** (MCP) is an open standard that gives AI assistants a live connection to external tools and data. Think of it like a USB port for AI — instead of pasting your docs into a chat window, your AI tool connects directly and can read, search, and act on your content in real time.

```curl
https://sstrader.readme.io/mcp
```

More details how to connect to this MCP Server can be found on [https://docs.readme.com/main/docs/mcp-servers](https://docs.readme.com/main/docs/mcp-servers)

## Skills
> This file is under development

## LLMs.txt
[LLMs.txt](https://llmstxt.org/) works as a configuration file at the root of your documentation site that AI language models can access and interpret. This [llms.txt](/llms.txt) file functions as a technical guide that instructs AI systems on how to properly read and reference your content.

<br />