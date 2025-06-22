# Claude Code Non-Interactive Blog Example

This example demonstrates using multiple context modules to create a technical blog post.

## Contexts Used

- [`blog-writing-context.md`](../../contexts/blog-writing-context.md) - Technical blog writing expertise
- [`non-interactive-claude-code-context.md`](../../contexts/non-interactive-claude-code-context.md) - Claude Code for scripting and automation

## Prompt Used

```
@blog-writing-context.md @non-interactive-claude-code-context.md 

Write a technical blog post about Claude Code's non-interactive features
```

## Output

The resulting blog post is available in [`claude-code-noninteractive-blog.md`](claude-code-noninteractive-blog.md).

This demonstrates how to combine domain-specific context (Claude Code) with process-specific context (blog writing).