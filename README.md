# AI Playbooks

A systematic framework for creating structured knowledge modules that provide AI agents with domain-specific expertise for complex tasks.

## What Are AI Playbooks?

AI Playbooks are self-contained knowledge modules designed specifically for AI agent consumption. Unlike tutorials or documentation written for humans, playbooks provide actionable expertise that AI agents can immediately apply without requiring repeated explanations of foundational concepts.

Each playbook follows a systematic framework with structured sections: **Key Concepts**, **Common Patterns**, **Implementation Details**, **Validation Methods**, and **Authoritative References**. This consistent structure enables AI agents to quickly locate and apply domain knowledge.

## Core Principle: Playbook as Subject, Action as Verb

Playbooks function as "subjects" that can be paired with any "verb" (action), similar to how REST APIs work with resources and HTTP methods. The playbook provides the domain knowledge, while your instruction specifies the action to perform.

```
# Standard interaction requires background explanation
You: "Help me set up Neovim with a package manager"
AI: ... (uses limited training knowledge) ...

# With playbook, AI has immediate domain expertise
You: "@neovim-lazy-playbook.md Set up Neovim with essential plugins for coding"
AI: "I'll configure Lazy.nvim with LSP, Telescope, Treesitter, and Neo-tree..."
```

### Same Knowledge, Different Actions

The same playbook supports multiple task types without requiring separate modules:

```
# Creation task
You: "@neovim-lazy-playbook.md Set up Neovim with essential plugins for coding"
AI: "I'll configure Lazy.nvim with LSP, Telescope, Treesitter, and Neo-tree..."

# Debugging task
You: "@neovim-lazy-playbook.md Debug why my Neovim plugins aren't loading"
AI: "Let's check your lazy-lock.json file and plugin configuration..."

# Optimization task
You: "@neovim-lazy-playbook.md Optimize my Neovim startup time"  
AI: "I'll analyze your plugin loading strategy and suggest lazy-loading improvements..."
```

## Composability: Combining Playbooks for Complex Tasks

Playbooks are designed to be composable. Multiple playbooks can be combined to tackle complex, multi-domain challenges:

```
# Multi-domain task combining blog writing expertise with technical tool knowledge
You: "@blog-writing-playbook.md @non-interactive-claude-code-playbook.md 
     Write a technical blog post about Claude Code's non-interactive features"
AI: "I'll write a structured technical blog post covering Claude Code's non-interactive 
     mode, with proper prerequisites, implementation examples, and validation steps..."
```

This composability enables sophisticated workflows by combining domain expertise from multiple areas. See [examples/claude-code-noninteractive-blog/](examples/claude-code-noninteractive-blog/) for a complete blog post created using this multi-playbook approach.

## Systematic Framework

Every playbook follows a consistent structure optimized for AI consumption:

### Required Sections
- **Key Concepts**: Essential domain knowledge, definitions, and core principles
- **Common Patterns**: Standard approaches, configurations, and decision frameworks  
- **Implementation Details**: Executable code examples, commands, and specific procedures
- **Validation Methods**: Success criteria, error handling, and troubleshooting patterns
- **Authoritative References**: Links to official documentation and primary sources

### AI-Optimized Content
- **Direct Information Delivery**: Facts first, minimal context
- **Pattern Recognition**: Multiple examples showing variations and use cases
- **Self-Contained**: Complete domain knowledge without external dependencies
- **Actionable**: Executable commands and code that work independently

## Available Playbooks

### 📚 Ready to Use
- **blog-writing-playbook.md** - Technical blog writing expertise with structured approaches
- **claude-code-commands-playbook.md** - Claude Code custom commands and automation workflows  
- **neovim-lazy-playbook.md** - Neovim configuration with Lazy package manager
- **neovim-markdown-editor-playbook.md** - Optimizing Neovim as a markdown editor
- **neovim-python-development-playbook.md** - Python development environment in Neovim
- **non-interactive-claude-code-playbook.md** - Claude Code for scripting and automation

### 🛠️ Meta Playbooks  
- **playbook-creation.md** - Complete framework for building authoritative playbooks

### 💡 Examples
- **examples/claude-code-noninteractive-blog/** - Blog post demonstrating multi-playbook composition

## Building Your Own Playbooks

All playbooks follow the systematic framework detailed in `playbooks/playbook-creation.md`. The framework emphasizes:

- **Authority-First Sources**: Official documentation, RFCs, and primary sources only
- **AI-Agent Optimization**: Structured for immediate AI consumption and application
- **Validation Testing**: Every playbook tested with AI agents before release
- **Composability**: Designed to work independently or combined with other playbooks

### Quick Start
1. Identify authoritative sources for your domain
2. Extract common patterns and implementation details
3. Structure content using the required sections
4. Test with AI agents for completeness and clarity
5. Validate all examples work independently

## Contributing

New playbooks should:
- Follow the framework in `playbooks/playbook-creation.md`
- Be fully self-contained with no external tutorial dependencies
- Focus on one domain or tool with deep coverage
- Include working code examples and validation methods
- Be tested with AI agents before submission

## Repository Structure

```
playbooks/         # All knowledge modules (meta and domain-specific)
examples/          # Practical demonstrations of multi-playbook usage
```

## License

Creative Commons Attribution 4.0 International License (CC BY 4.0)