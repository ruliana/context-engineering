# Context Engineering

A collection of structured knowledge modules designed to provide AI agents with domain-specific expertise for complex tasks.

**Note:** "Context Engineering" is not an established term in the field. I propose it as a natural progression from prompt engineering - moving from crafting individual prompts to orchestrating comprehensive knowledge contexts. More thoughts on this progression can be found in "[From Prompt Engineering to Context Engineering](https://claude.ai/public/artifacts/b0816f4a-9d45-4227-ae18-91bb1b60fcc1)".

This repository contains self-contained knowledge modules that can be loaded into AI conversations to provide immediate domain expertise without requiring repeated explanations of foundational concepts.

## Example Usage

```
# Standard interaction requires background explanation
You: "Help me set up Neovim with a package manager"
AI: ... (uses the knowledge available in the internet at training time) ...

# With context module, AI has immediate domain knowledge
You: "@neovim-lazy-context.md Set up Neovim with essential plugins for coding"
AI: "I'll configure Lazy.nvim with LSP, Telescope, Treesitter, and Neo-tree..."
```

### Multi-Context Example

```
# Combining multiple contexts for complex tasks
You: "@blog-writing-context.md @non-interactive-claude-code-context.md 
     Write a technical blog post about Claude Code's non-interactive features"
AI: "I'll write a structured technical blog post covering Claude Code's non-interactive 
     mode, with proper prerequisites, implementation examples, and validation steps..."
```

These modules provide comprehensive domain knowledge rather than simple prompt templates. See the [claude-code-noninteractive-blog.md](examples/claude-code-noninteractive-blog/claude-code-noninteractive-blog.md) for the complete blog post created using this two-context approach.

## Context as Subject, Action as Verb

Context modules function as "subjects" that can be paired with any "verb" (action), similar to how REST APIs work with resources and HTTP methods. The context provides the domain knowledge, while your instruction specifies the action to perform.

```
# Same context, different actions
You: "@neovim-lazy-context.md Set up Neovim with essential plugins for coding"
AI: "I'll configure Lazy.nvim with LSP, Telescope, Treesitter, and Neo-tree..."

You: "@neovim-lazy-context.md Debug why my Neovim plugins aren't loading"
AI: "Let's check your lazy-lock.json file and plugin configuration..."

You: "@neovim-lazy-context.md Optimize my Neovim startup time"
AI: "I'll analyze your plugin loading strategy and suggest lazy-loading improvements..."
```

Context modules contain foundational knowledge but do not include action-specific instructions like "create", "fix", or "evaluate". Instead, they provide the expertise that can be applied to any action within that domain.

## Module Structure

Each context module contains:

- **Key Concepts**: Core principles and definitions
- **Common Patterns**: Standard approaches and decision frameworks  
- **Implementation Details**: Code examples, commands, and procedures
- **Validation Methods**: Success criteria and error handling
- **Authoritative References**: Links to official documentation

Modules focus on essential knowledge structured for AI consumption, not step-by-step tutorials.

## Usage Patterns

Context modules provide foundational knowledge that supports multiple task types. For example, `neovim-lazy-context.md` enables:

- **Creation**: "Create a Neovim configuration using Lazy package manager"
- **Debugging**: "Help debug issues in my Neovim Lazy setup"
- **Optimization**: "Optimize my existing Neovim configuration for performance"
- **Evaluation**: "Review my Neovim setup and suggest improvements"

The same knowledge base supports different actions without requiring separate context modules for each task type.

## Available Modules

### 📚 Ready to Use
- `blog-writing-context.md` - Technical blog writing expertise
- `claude-code-commands-context.md` - Claude Code custom commands and workflows  
- `neovim-lazy-context.md` - Neovim with Lazy package manager
- `neovim-markdown-editor-context.md` - Neovim as markdown editor
- `non-interactive-claude-code-context.md` - Claude Code for scripting and automation

### 🛠️ Meta Modules  
- `context-creation.md` - How to build your own context modules

### 💡 Examples
- `claude-code-noninteractive-blog/` - Blog post created by combining multiple contexts

## Contributing

New context modules should:
- Be fully self-contained (no external dependencies)
- Focus on one domain or tool
- Include practical examples and code
- Test with AI agents before submitting

## Repository Structure

```
contexts/          # All knowledge modules (meta, generic, and domain-specific)
examples/          # Practical usage demonstrations
```

## License

MIT
