# Non-Interactive Claude Code Context

## Key Concepts

- **Non-Interactive Mode**: Claude Code execution mode using the `-p` flag that processes a single query and exits without starting an interactive session
- **Print Mode**: Alternative term for non-interactive mode, emphasizing the immediate output behavior
- **Query Processing**: Single-turn execution where Claude processes input and returns output without maintaining conversation state
- **Automation Context**: Usage pattern where Claude Code is integrated into scripts, CI/CD pipelines, or other automated workflows
- **Piping Support**: Ability to receive input from stdin and send output to stdout for integration with other command-line tools
- **Output Formats**: Structured response formats (text, json, stream-json) for programmatic consumption
- **Max Turns**: Limit on conversation rounds to prevent infinite processing in automated contexts

## Common Patterns

### Basic Query Pattern
```bash
claude -p "query text here"
```
**When to use**: Simple one-off questions or tasks that don't require file interaction
**Expected output**: Direct text response followed by command termination

### File Input Pattern
```bash
cat file.txt | claude -p "analyze this file"
```
**When to use**: Processing file content without modifying the original file
**Expected output**: Analysis or response based on piped file content

### Structured Output Pattern
```bash
claude -p "list project dependencies" --output-format json
```
**When to use**: When response needs to be parsed by other tools or scripts
**Expected output**: JSON-formatted response suitable for programmatic processing

### Script Integration Pattern
```bash
#!/bin/bash
result=$(claude -p "summarize recent git commits")
echo "Summary: $result"
```
**When to use**: Embedding Claude Code in automation scripts
**Expected output**: Command substitution captures Claude's response for further processing

### Multi-line Query Pattern
```bash
claude -p "$(cat <<'EOF'
Review this code for potential issues:
- Security vulnerabilities
- Performance problems  
- Code quality concerns
EOF
)"
```
**When to use**: Complex queries that require multiple lines or structured input
**Expected output**: Detailed analysis addressing each specified concern

## Implementation Details

### Basic Non-Interactive Execution
1. `claude -p "what technologies does this project use?"`
2. Claude analyzes project context and responds
3. Command exits after response

**Validation**: Command returns exit code 0 on success
**Common errors**: `Error: No API key found` = Set ANTHROPIC_API_KEY environment variable

### File Processing Workflow
1. `cat requirements.txt | claude -p "explain these dependencies"`
2. File content is piped to Claude for analysis
3. Response explains each dependency's purpose
4. Command exits with analysis output

**Validation**: Response contains information about piped file content
**Common errors**: `Error: empty input` = Check file exists and has content

### JSON Output Configuration
1. `claude -p "list main functions in this codebase" --output-format json`
2. Claude processes request and formats response as JSON
3. JSON structure suitable for parsing with jq or similar tools

**Validation**: Output is valid JSON (test with `| jq .`)
**Common errors**: `Invalid format` = Use supported formats: text, json, stream-json

### Verbose Logging Setup
1. `claude -p "debug this error" --verbose`
2. Enhanced logging shows processing details
3. Helpful for troubleshooting automation issues

**Validation**: Additional debug information appears in output
**Common errors**: Log level errors = Check if verbose flag is correctly specified

### Turn Limiting for Automation
1. `claude -p "analyze and fix issues" --max-turns 1`
2. Prevents extended back-and-forth in automated contexts
3. Ensures predictable execution time

**Validation**: Process completes within specified turn limit
**Common errors**: Task incomplete = Increase max-turns or simplify query

## Troubleshooting

### Authentication Issues
**Symptoms**: `Error: No API key found` or `Authentication failed`
**Cause**: Missing or invalid ANTHROPIC_API_KEY environment variable
**Fix**: 
```bash
export ANTHROPIC_API_KEY="your-api-key-here"
claude -p "test query"
```

### Empty or Invalid Input
**Symptoms**: `Error: empty input` or no response
**Cause**: Piped input is empty or query string is malformed
**Fix**: 
```bash
# Check file has content
ls -la file.txt
# Verify pipe works
cat file.txt | head -n 5
# Test with simple query
claude -p "hello"
```

### JSON Format Errors  
**Symptoms**: `Invalid format` or malformed JSON output
**Cause**: Incorrect output format specification
**Fix**:
```bash
# Use exact format names
claude -p "query" --output-format json
# Validate JSON output
claude -p "query" --output-format json | jq .
```

### Script Integration Failures
**Symptoms**: Script hangs or produces unexpected results
**Cause**: Interactive mode triggered instead of print mode
**Fix**:
```bash
# Ensure -p flag is used
claude -p "query" # not just claude "query"
# Add explicit exit handling
result=$(claude -p "query" || echo "error")
```

### Project Context Issues
**Symptoms**: Claude doesn't understand project structure
**Cause**: Running outside project directory or no project files
**Fix**:
```bash
# Run from project root
cd /path/to/project
claude -p "analyze project"
# Or specify explicit context
claude -p "analyze files in current directory"
```

### Output Parsing Problems
**Symptoms**: Unexpected characters or formatting in output
**Cause**: Mixed output formats or verbose logging interfering
**Fix**:
```bash
# Use clean output format
claude -p "query" --output-format text
# Avoid verbose in automated scripts
claude -p "query" # without --verbose
```

### Performance and Timeout Issues
**Symptoms**: Command hangs or takes too long
**Cause**: Complex queries or network issues
**Fix**:
```bash
# Simplify query
claude -p "brief summary of main.py"
# Add timeout wrapper
timeout 60s claude -p "complex analysis query"
```

## Authoritative References

- [Claude Code CLI Reference](https://docs.anthropic.com/en/docs/claude-code/cli-reference) - Official command-line interface documentation
- [Claude Code Quickstart](https://docs.anthropic.com/en/docs/claude-code/quickstart) - Getting started guide with usage patterns
- [Claude Code Overview](https://docs.anthropic.com/en/docs/claude-code/overview) - Installation and basic usage instructions