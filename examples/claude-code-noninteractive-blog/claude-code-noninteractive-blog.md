# Turn Your Scripts Into Smart Automation with Claude Code Non-Interactive Mode

If you're using Cursor or other AI-powered development tools, you've probably experienced the magic of having AI understand your code context and make intelligent suggestions. But what if you could bring that same intelligence to your command-line scripts and automation workflows?

Enter Claude Code's non-interactive mode—a powerful feature that transforms Claude from a chat companion into a scriptable AI decision-maker for your automation tasks.

*This article demonstrates context engineering in action: it was created by composing two context modules - `claude-code-context.md` and `blog-writing-context.md` - to provide both domain expertise and writing framework.*

## The Problem with Traditional Automation

Most automation scripts follow rigid if-then logic. Your download cleanup script might sort files by extension, your git workflow might follow predetermined rules, and your code analysis tools output the same format regardless of context.

But real-world scenarios are messy. That PDF in your downloads folder could be an important document, a random article, or complete junk. Your git commit might need a detailed message or just a simple fix note. Traditional scripts can't make these nuanced decisions.

## The Solution: AI-Powered Decision Making in Scripts

Claude Code's non-interactive mode (using the `-p` flag) lets you embed intelligent decision-making directly into your automation workflows. Instead of hardcoded rules, your scripts can ask Claude to analyze context and make smart choices.

## Prerequisites

Before we dive in, you'll need:
- Claude Code installed (`npm install -g @anthropics/claude-code` or similar)
- `ANTHROPIC_API_KEY` environment variable set
- Basic shell scripting knowledge
- Familiarity with command-line tools and piping

## Real-World Example: Intelligent Downloads Cleanup

Let's start with a practical example—cleaning up your downloads folder using AI to decide what to do with each file.

### Basic Smart Cleanup Script

```bash
#!/bin/bash

DOWNLOADS_DIR="$HOME/Downloads"
ARCHIVE_DIR="$HOME/Documents/Archive"
TRASH_DIR="$HOME/.Trash"

for file in "$DOWNLOADS_DIR"/*; do
    if [ -f "$file" ]; then
        filename=$(basename "$file")
        
        # Ask Claude to analyze the file and decide what to do
        decision=$(claude -p "Analyze this filename: '$filename'. 
        Respond with exactly one word:
        - KEEP (if it looks important/useful)
        - ARCHIVE (if it's a document worth keeping but not urgent)  
        - DELETE (if it looks like junk/temporary)")
        
        case "$decision" in
            "KEEP")
                echo "✅ Keeping: $filename"
                ;;
            "ARCHIVE")
                mv "$file" "$ARCHIVE_DIR/"
                echo "📁 Archived: $filename"
                ;;
            "DELETE")
                mv "$file" "$TRASH_DIR/"
                echo "🗑️  Deleted: $filename"
                ;;
            *)
                echo "❓ Unknown decision for: $filename (got: $decision)"
                ;;
        esac
    fi
done
```

**What this does**: Loops through your downloads, asks Claude to analyze each filename, and takes action based on AI recommendations.

**Expected output**: Files get sorted intelligently—Claude might keep "tax_documents_2024.pdf" but delete "random_meme.jpg".

### Enhanced Version with File Content Analysis

For better decisions, let's analyze actual file content:

```bash
#!/bin/bash

analyze_and_organize() {
    local file="$1"
    local filename=$(basename "$file")
    local extension="${filename##*.}"
    
    # Create a detailed prompt with file info
    local prompt="Analyze this file for organization:
    Filename: $filename
    Size: $(ls -lh "$file" | awk '{print $5}')
    Type: $extension file
    
    Based on the filename and file type, categorize as:
    - DOCUMENTS (important docs, PDFs, etc.)
    - MEDIA (photos, videos worth keeping)
    - CODE (source files, configs)
    - TEMP (downloads, installers, junk)
    
    Respond with just the category name."
    
    local category=$(claude -p "$prompt")
    
    case "$category" in
        "DOCUMENTS")
            mkdir -p "$HOME/Documents/ImportantFiles"
            mv "$file" "$HOME/Documents/ImportantFiles/"
            echo "📄 Document filed: $filename"
            ;;
        "MEDIA")
            mkdir -p "$HOME/Pictures/FromDownloads"
            mv "$file" "$HOME/Pictures/FromDownloads/"
            echo "🖼️  Media saved: $filename"
            ;;
        "CODE")
            mkdir -p "$HOME/Code/Downloads"
            mv "$file" "$HOME/Code/Downloads/"
            echo "💻 Code archived: $filename"
            ;;
        "TEMP")
            mv "$file" "$HOME/.Trash/"
            echo "🗑️  Cleaned up: $filename"
            ;;
        *)
            echo "❓ Unclear category for: $filename"
            ;;
    esac
}

# Process all files in downloads
for file in "$HOME/Downloads"/*; do
    [ -f "$file" ] && analyze_and_organize "$file"
done
```

## More Smart Automation Patterns

### Intelligent Git Commit Messages

```bash
#!/bin/bash

# Generate contextual commit messages based on changes
generate_commit_message() {
    local changes=$(git diff --cached --name-only | head -10)
    local diff_sample=$(git diff --cached | head -20)
    
    local message=$(claude -p "Based on these staged changes:
    Files: $changes
    
    Sample diff:
    $diff_sample
    
    Generate a concise git commit message (under 50 chars) that describes what was changed.")
    
    echo "$message"
}

# Stage changes and generate smart commit message
git add .
commit_msg=$(generate_commit_message)
echo "Suggested commit: $commit_msg"
read -p "Use this message? (y/n): " -n 1 -r
if [[ $REPLY =~ ^[Yy]$ ]]; then
    git commit -m "$commit_msg"
fi
```

### Smart Log Analysis

```bash
#!/bin/bash

# Analyze log files and extract insights
analyze_logs() {
    local log_file="$1"
    
    # Get recent errors and let Claude analyze them
    local recent_errors=$(tail -100 "$log_file" | grep -i error | head -10)
    
    if [ -n "$recent_errors" ]; then
        claude -p "Analyze these recent log errors and summarize:
        $recent_errors
        
        Provide:
        1. Most critical issue (one line)
        2. Recommended action (one line)"
    else
        echo "No recent errors found in $log_file"
    fi
}

# Check multiple log files
for log in /var/log/*.log; do
    [ -r "$log" ] && echo "=== $(basename "$log") ===" && analyze_logs "$log"
done
```

### Intelligent Code Review Helper

```bash
#!/bin/bash

# Smart code review for pull requests
review_changes() {
    local files_changed=$(git diff --name-only HEAD~1)
    
    for file in $files_changed; do
        if [[ "$file" =~ \.(js|py|go|rs)$ ]]; then
            local file_diff=$(git diff HEAD~1 "$file")
            
            echo "=== Reviewing $file ==="
            claude -p "Review this code change for:
            - Potential bugs
            - Security issues  
            - Performance concerns
            
            Code diff:
            $file_diff
            
            Provide brief, actionable feedback:"
        fi
    done
}

review_changes
```

## Advanced Patterns with JSON Output

For more complex workflows, use Claude's JSON output for structured data:

```bash
#!/bin/bash

# Analyze project structure and get structured recommendations
analyze_project() {
    local files=$(find . -name "*.js" -o -name "*.py" -o -name "*.md" | head -20)
    
    local analysis=$(claude -p "Analyze this project structure:
    $files
    
    Return JSON with:
    {
        \"project_type\": \"description\",
        \"main_language\": \"language\",
        \"suggestions\": [\"suggestion1\", \"suggestion2\"],
        \"missing_files\": [\"file1\", \"file2\"]
    }" --output-format json)
    
    # Parse JSON and take actions
    echo "$analysis" | jq -r '.suggestions[]' | while read suggestion; do
        echo "💡 Suggestion: $suggestion"
    done
    
    echo "$analysis" | jq -r '.missing_files[]' | while read missing; do
        echo "⚠️  Missing: $missing"
    done
}
```

## Best Practices for Smart Scripts

### 1. Design Clear Prompts

```bash
# Good: Specific, constrained output
decision=$(claude -p "Categorize '$filename' as: KEEP, ARCHIVE, or DELETE. One word only.")

# Bad: Open-ended, unpredictable output  
decision=$(claude -p "What should I do with this file?")
```

### 2. Handle Edge Cases

```bash
# Always validate Claude's response
case "$decision" in
    "KEEP"|"ARCHIVE"|"DELETE")
        # Handle expected responses
        ;;
    *)
        echo "Unexpected response: $decision"
        # Default action or manual review
        ;;
esac
```

### 3. Add Confirmation for Destructive Actions

```bash
if [ "$decision" = "DELETE" ]; then
    read -p "Delete $filename? (y/n): " -n 1 -r
    [[ $REPLY =~ ^[Yy]$ ]] && rm "$file"
fi
```

### 4. Use Timeouts for Automation

```bash
# Prevent hanging in automated environments
decision=$(timeout 30s claude -p "$prompt" || echo "TIMEOUT")
```

## Troubleshooting Common Issues

### Authentication Problems
```bash
# Test your setup
if ! claude -p "test" >/dev/null 2>&1; then
    echo "Claude Code not working. Check ANTHROPIC_API_KEY"
    exit 1
fi
```

### Handling Rate Limits
```bash
# Add delays between requests in loops
for file in "$@"; do
    process_file "$file"
    sleep 1  # Respect rate limits
done
```

### Debugging Responses
```bash
# Use verbose mode for troubleshooting
claude -p "$prompt" --verbose
```

## What's Next

Once you've mastered basic smart automation, try these advanced patterns:

- **Smart deployment scripts** that analyze code changes and choose deployment strategies
- **Intelligent monitoring** that explains unusual metrics patterns  
- **Context-aware backup scripts** that prioritize files based on content analysis
- **Smart dependency management** that suggests updates based on your usage patterns

The key insight is that Claude Code's non-interactive mode transforms any script from a dumb automaton into an intelligent assistant that can make nuanced decisions based on context.

Start with simple file organization like the downloads example, then gradually add AI decision-making to your existing automation workflows. You'll be surprised how much smarter your scripts can become with just a few lines of Claude integration.

---

*Ready to make your scripts smarter? Try the downloads cleanup script above and see how Claude's intelligent decision-making compares to your usual file sorting rules.*

## Context Engineering Notes

This article was created using context engineering principles:

1. **Context Composition**: Combined `claude-code-context.md` (technical domain knowledge) with `blog-writing-context.md` (writing framework and best practices)

2. **Layered Expertise**: The Claude Code context provided deep technical knowledge about non-interactive mode, while the blog writing context provided structure, tone, and optimization guidance

3. **Composable Knowledge**: Each context module is self-contained and reusable - the blog writing context can be applied to any technical topic, while the Claude Code context can inform different content formats

This demonstrates how context engineering enables rapid, high-quality content creation by mixing domain expertise with structural frameworks.