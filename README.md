# Context Engineering

A curated collection of composable context modules designed to rapidly equip AI agents with domain-specific knowledge and frameworks for performing complex tasks.

## Overview

Context engineering is a specialization of Retrieval-Augmented Generation (RAG) focused on crafting structured, composable context that can be mixed and matched to give AI agents precisely the knowledge they need for specific tasks.

**Note:** "Context Engineering" is not yet a term in the field, but I believe it is the natural next step of prompt engineering (some thoughts about it on "[From Prompt Engineering to Context Engineering](https://claude.ai/public/artifacts/b0816f4a-9d45-4227-ae18-91bb1b60fcc1)" - the progression from prompt crafting to context orchestration.

This repository provides self-contained context modules that can be referenced in AI conversations, loaded as initial context, or combined to build comprehensive knowledge bases for AI agents. While optimized for AI consumption, these documents also serve as excellent references for humans.

**Note:** This is not a prompt library. These are knowledge modules designed to provide deep, actionable context about specific domains, tools, or methodologies.

## Context Engineering Principles

Context varies from **generic** to **specific**:

- **Generic contexts**: Universal knowledge applicable across domains (e.g., how to write a blog post, project management principles)
- **Specific contexts**: Specialized knowledge for particular tools or domains (e.g., BigQuery Pipe Syntax, Racket Programming Language, in-house system architectures)

Generic contexts help prime models with established best practices, while specific contexts fill knowledge gaps for new technologies or proprietary systems that lack training data.

## Context Modules

Context modules are self-contained knowledge documents optimized for AI consumption. They range from **generic** to **specific**:

- **Generic contexts**: Universal knowledge applicable across domains (e.g., blog writing, project management principles)
- **Specific contexts**: Specialized knowledge for particular tools or domains (e.g., Claude Code workflows, database optimization, API design)
- **Meta contexts**: Knowledge about creating and structuring other contexts (e.g., context creation methodology)

Context modules are designed to be **composable** - you can combine multiple modules for complex, multi-domain tasks. For example, creating a technical blog post about Docker might use both "blog-writing-context" and "docker-context" modules.

## Usage Patterns

1. **Single Context**: Load one context module for focused, domain-specific tasks
2. **Context Composition**: Combine multiple contexts for complex, multi-domain scenarios
3. **Meta + Domain**: Use meta contexts to guide creation, then apply domain contexts
4. **Iterative Refinement**: Start with generic contexts, then layer in specific domain knowledge

## Contributing

When adding new context modules:
- **Meta contexts**: Focus on teaching principles and methodologies
- **Domain contexts**: Provide actionable, domain-specific knowledge
- Ensure each document is fully self-contained
- Test with AI agents to verify effectiveness
- Design for composability with other modules

## Module Index

### 📚 Context Modules
- `context-creation.md` - How to build effective context modules
- `blog-writing-context.md` - Technical blog writing expertise
- `neovim-lazy-context.md` - Neovim with Lazy package manager
- `neovim-markdown-editor-context.md` - Neovim as markdown editor
- `non-interactive-claude-code-context.md` - Claude Code usage patterns and workflows for non-interactive use (`claude -p`)
- `claude-code-commands-context.md` - Claude Code custom commands creation and usage patterns

### 💡 Examples
- `claude-code-noninteractive-blog/` - Blog post created by combining `non-interactive-claude-code-context.md` and `blog-writing-context.md`

## Repository Structure

```
contexts/          # All knowledge modules (meta, generic, and domain-specific)
examples/          # Practical usage demonstrations
```

## License

MIT
