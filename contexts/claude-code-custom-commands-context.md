# Claude Code Custom Commands Context

## Key Concepts

- **Custom Slash Command**: Reusable prompt template stored as Markdown file in `.claude/commands/` directory
- **Project Commands**: Commands stored in project's `.claude/commands/` directory, shared with team (prefix: `/project:`)
- **Personal Commands**: Commands stored in `~/.claude/commands/` directory, available across all projects (prefix: `/user:`)
- **Arguments Placeholder**: `$ARGUMENTS` keyword in command files that gets replaced with parameters passed to command
- **File References**: `@filename` syntax to include file contents in command context
- **Bash Integration**: `!command` prefix to execute bash commands before running the slash command
- **YAML Frontmatter**: Optional metadata at top of command files for configuration and documentation
- **Command Namespacing**: Organize commands in subdirectories for complex project structures

## Common Patterns

### Basic Command Structure
```markdown
# Command Name

Brief description of what this command does.

## Instructions

- Directive to follow
- Another directive to follow
- Practices to avoid

## Process

1. Specific action to take
   - Expected outcome or instruction to follow
2. Next action

## Validation

- Confirm all specified directives were followed
- Verify expected outcomes were achieved
- Check that no unintended side effects occurred
```

### Parameterized Command with Arguments
```markdown
# Fix GitHub Issue

Analyze and fix the specified GitHub issue systematically.

Please analyze and fix the GitHub issue: $ARGUMENTS

## Instructions

- Be pragmatic and precise. Do not implement anything more than fix the issue
- Use the TDD red-green cycle (one test, one implementation, one test, one implementation)

## Process

Follow the instructions below sistematically.

1. Use `gh issue view $ARGUMENTS` to get issue details
2. Search relevant files using grep/find
3. Search relevant PRs using `gh`
4. Run test suite to make sure you can run it and all tests are passing
4. Write a unit test that demonstrates the issue
   - Do not proceed until you have a failing test
5. Implement changes following project conventions
6. Run test suite and linting
7. Create descriptive commit message
8. Push changes and create pull request

## Validation

1. Compare the original issue with the results. Check if everything was covered, no more, no less.
2. Run all tests, check if there is no regressions
```

### Multi-Argument Command
```markdown
# Create Feature Branch

Create a new feature branch with comprehensive setup.

Feature details: $ARGUMENTS

## Arguments

Parse the arguments as: feature-type feature-name [base-branch]
- Extract feature type from first word of $ARGUMENTS
- Extract feature name from second word of $ARGUMENTS
- Use third word as base branch if provided, otherwise use main

## Instructions

- Create branch following naming convention: `{feature-type}/{feature-name}`
- Ensure clean development environment setup
- Verify all dependencies and tests work correctly

## Process

1. **Branch Management**
   - Ensure we're on the correct base branch
   - Create branch: `{feature-type}/{feature-name}`
   - Push branch to remote with upstream tracking

2. **Setup Development Environment**
   - Install/update dependencies if needed
   - Run initial tests to ensure clean state
   - Create basic file structure if needed

## Validation

- Confirm branch exists: `git branch -a | grep {feature-type}/{feature-name}`
- Verify upstream tracking: `git branch -vv`
- Check clean working directory: `git status`
- Validate tests pass: run test suite
- Ensure dependencies are current: check package manager status
```

### Multi-Step Workflow Command
```markdown
# Code Review Checklist

Perform comprehensive code review following team standards.

Team standards file: `a-file-with-team-standards.md`

## Review Criteria

1. **Code Quality**
   - Check for consistent formatting
   - Verify proper error handling
   - Ensure appropriate comments/documentation

2. **Security Assessment**
   - Validate input sanitization
   - Check for hardcoded secrets
   - Review authentication/authorization

3. **Performance Analysis**
   - Identify potential bottlenecks
   - Check for memory leaks
   - Evaluate algorithm efficiency

4. **Testing Coverage**
   - Verify unit tests exist
   - Check integration test coverage
   - Validate edge case handling

## Validation

- Confirm all review criteria were addressed
- Verify actionable feedback was provided for each issue
- Check that security and performance concerns were identified
- Ensure recommendations are specific and implementable
```

### File-Reference Command
```markdown
# Analyze Configuration

Review project configuration files for best practices.

Please analyze these configuration files:
- @package.json
- @tsconfig.json
- @.eslintrc.js

## Analysis Focus
- Dependencies and version compatibility
- Build and lint configuration
- Security settings and best practices

## Validation

- Confirm all referenced files were analyzed
- Verify compatibility issues were identified
- Check that security recommendations are actionable
- Ensure configuration suggestions follow best practices
```

### Bash-Integrated Command
```markdown
# Environment Setup

Set up development environment with current system info.

!uname -a
!node --version
!npm --version

Based on the system information above, help me:
1. Verify all required dependencies are installed
2. Set up the development environment
3. Configure any missing tools or settings

## Validation

- Confirm system information was captured correctly
- Verify all dependencies are compatible with current system
- Check that setup instructions are system-specific
- Ensure configuration changes were applied successfully
```

## Implementation Details

### Creating Project Commands

1. **Create Commands Directory**
```bash
mkdir -p .claude/commands
```

2. **Create Command File**
```bash
# Create a new command file
touch .claude/commands/optimize-performance.md
```

3. **Add Command Content**
```markdown
# Optimize Performance

Analyze code for performance bottlenecks and suggest improvements.

## Analysis Process

1. **Profile Current Performance**
   - Run existing benchmarks
   - Identify slow operations
   - Measure memory usage

2. **Optimization Strategies**
   - Database query optimization
   - Caching implementation
   - Algorithm improvements
   - Resource usage reduction

3. **Validation**
   - Run performance tests
   - Compare before/after metrics
   - Ensure functionality preserved
```

### Creating Personal Commands

1. **Create Global Commands Directory**
```bash
mkdir -p ~/.claude/commands
```

2. **Create Cross-Project Command**
```bash
# Example: debugging helper command
cat > ~/.claude/commands/debug-issue.md << 'EOF'
# Debug Issue

Systematic debugging approach for any codebase.

Issue to debug: $ARGUMENTS

## Debugging Steps

1. **Reproduce the Issue**
   - Create minimal test case
   - Document exact steps
   - Capture error messages/logs

2. **Isolate the Problem**
   - Check recent changes
   - Test in clean environment
   - Verify dependencies

3. **Implement Solution**
   - Fix root cause
   - Add error handling
   - Update documentation

4. **Validate Fix**
   - Test fix thoroughly
   - Run regression tests
   - Update test coverage
EOF
```

### Command Organization Patterns

**Simple Structure**
```
.claude/
└── commands/
    ├── code-review.md
    ├── optimize.md
    └── test-setup.md
```

**Organized Structure**
```
.claude/
└── commands/
    ├── development/
    │   ├── feature-branch.md
    │   └── hotfix.md
    ├── testing/
    │   ├── unit-tests.md
    │   └── integration-tests.md
    └── deployment/
        ├── staging.md
        └── production.md
```

### YAML Frontmatter Configuration

```markdown
---
description: "Automated security audit for codebase"
category: "security"
version: "1.0"
author: "Security Team"
requires:
  - "bandit"
  - "safety"
  - "semgrep"
---

# Security Audit

Comprehensive security analysis of the codebase.

!bandit -r . -f json
!safety check
!semgrep --config=auto .

Based on the security scan results above:

1. **Critical Issues**
   - Review and fix high-severity vulnerabilities
   - Update vulnerable dependencies

2. **Recommendations**
   - Implement additional security measures
   - Add security tests to CI/CD pipeline
```

## Validation Methods

### Testing Command Functionality

1. **Invoke Command**
```bash
# Test command with arguments
/project:fix-github-issue 123

# Test command without arguments
/project:code-review
```

2. **Verify Expected Behavior**
- Command executes without errors
- Arguments are properly substituted
- File references load correctly
- Bash commands execute successfully

### Common Error Patterns

**Error**: Command not found
**Symptoms**: `/project:mycommand` shows "Command not found"
**Cause**: File not in `.claude/commands/` or incorrect naming
**Fix**: Verify file location and use kebab-case naming

**Error**: Arguments not substituted
**Symptoms**: `$ARGUMENTS` appears literally in output
**Cause**: Missing arguments when invoking command
**Fix**: Provide arguments: `/project:command arg1 arg2`

**Error**: File reference fails
**Symptoms**: `@filename` shows file not found
**Cause**: Referenced file doesn't exist or wrong path
**Fix**: Use correct relative paths from project root

**Error**: Bash command fails
**Symptoms**: `!command` shows execution error
**Cause**: Command not available or incorrect syntax
**Fix**: Test bash command independently first

### Command Effectiveness Validation

**Test Prompt**: "Using the custom command `/project:code-review`, perform a comprehensive review of the main application file."

**Success Criteria**:
- Command executes without requesting additional context
- Follows all specified review criteria
- Provides actionable feedback
- Completes workflow steps systematically

## Best Practices

### Prompt Engineering for Commands

**Use Specific, Action-Oriented Language**
```markdown
# ❌ Vague
Help me with testing.

# ✅ Specific
Create comprehensive unit tests for the UserService class, including edge cases for authentication, data validation, and error handling.
```

**Include Context and Constraints**
```markdown
# ✅ Well-Contextualized Command
# Database Migration

Create a new database migration following our team's conventions.

Migration purpose: $ARGUMENTS

## Requirements
- Use sequelize migration format
- Include both up and down methods
- Add appropriate indexes
- Follow naming convention: YYYYMMDD-description.js
- Test migration in development environment before committing
```

**Leverage Extended Thinking**
```markdown
# Performance Analysis

Think deeply about the performance characteristics of this code: $ARGUMENTS

## Analysis Framework
1. **Complexity Analysis** - Think harder about algorithmic complexity
2. **Resource Usage** - Think more about memory and CPU usage patterns  
3. **Optimization Opportunities** - Think longer about potential improvements
4. **Trade-offs** - Consider the implications of each optimization
```

### Command Design Principles

**Single Responsibility**
- Each command should focus on one specific workflow
- Avoid combining unrelated tasks in a single command
- Create separate commands for different contexts

**Reusability**
- Design commands to work across different parts of the codebase
- Use parameters to make commands flexible
- Avoid hardcoded values specific to one use case

**Team Collaboration**
- Include clear descriptions and usage examples
- Follow consistent formatting and structure
- Document command purpose and expected outcomes

**Progressive Enhancement**
```markdown
# Basic Version
Review this code for obvious issues.

# Enhanced Version
# Code Review Protocol

Perform systematic code review following team standards.

Code to review: $ARGUMENTS

## Review Checklist

### 1. Functionality
- [ ] Code accomplishes intended purpose
- [ ] Edge cases are handled appropriately
- [ ] Error conditions are managed properly

### 2. Code Quality
- [ ] Follows project coding standards
- [ ] Appropriate abstractions and patterns
- [ ] Clear, readable, and maintainable

### 3. Security
- [ ] Input validation implemented
- [ ] No exposed sensitive data
- [ ] Authentication/authorization correct

### 4. Performance
- [ ] Efficient algorithms and data structures
- [ ] No obvious performance bottlenecks
- [ ] Appropriate caching strategies

### 5. Testing
- [ ] Adequate test coverage
- [ ] Tests cover edge cases
- [ ] Tests are maintainable

## Recommendations
Provide specific, actionable feedback for each issue found.
```

### Memory vs Commands Strategy

**Use Memory For:**
- Project-wide coding standards and conventions
- Team preferences and guidelines
- Consistent formatting and style rules
- Context that applies to all tasks

**Use Commands For:**
- Specific, repeatable workflows
- Multi-step procedures
- Templated responses
- Task-specific checklists

### Command Maintenance

**Version Control**
- Check commands into git repository
- Use meaningful commit messages for command changes
- Document command updates in team communications

**Regular Review**
- Periodically review command effectiveness
- Update commands based on workflow changes
- Remove or consolidate unused commands

**Documentation**
- Maintain a commands directory or README
- Include usage examples and prerequisites
- Document command interactions and dependencies

## Troubleshooting

### Command Not Loading
**Symptoms**: Command doesn't appear in slash menu
**Cause**: File not in correct directory or invalid format
**Fix**: 
1. Verify file is in `.claude/commands/` directory
2. Check file has `.md` extension
3. Ensure valid markdown format

### Arguments Not Working
**Symptoms**: `$ARGUMENTS` not replaced with actual values
**Cause**: Command invoked without parameters
**Fix**: Always provide arguments when using parameterized commands
```bash
# Wrong
/project:fix-issue

# Correct  
/project:fix-issue 123
```

### File References Failing
**Symptoms**: `@filename` shows "file not found"
**Cause**: Incorrect file path or file doesn't exist
**Fix**: Use paths relative to project root
```markdown
# Wrong
@src/components/Button.js

# Correct (if running from project root)
@src/components/Button.js
```

### Bash Commands Not Executing
**Symptoms**: `!command` fails or shows permission error
**Cause**: Command not available in PATH or insufficient permissions
**Fix**: 
1. Test command in terminal first
2. Use full path if needed: `!/usr/bin/git status`
3. Check file permissions for executable commands

### Command Too Generic
**Symptoms**: Command produces vague or unhelpful responses
**Cause**: Insufficient context or unclear instructions
**Fix**: Add specific requirements, examples, and success criteria
```markdown
# ❌ Too Generic
Review this code.

# ✅ Specific and Actionable
# API Endpoint Review

Review the API endpoint code: $ARGUMENTS

## Review Focus
1. **Security**: Validate authentication, authorization, input sanitization
2. **Performance**: Check for N+1 queries, inefficient operations
3. **Error Handling**: Verify proper HTTP status codes and error responses
4. **Documentation**: Ensure API documentation matches implementation

## Output Format
- List specific issues found
- Provide code snippets for fixes
- Rate overall code quality (1-10)
- Suggest priority for addressing issues
```

## Authoritative References

- [Claude Code Slash Commands - Official Documentation](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Claude Code Settings Configuration](https://docs.anthropic.com/en/docs/claude-code/settings)
- [Claude Code Common Workflows](https://docs.anthropic.com/en/docs/claude-code/common-workflows)
- [Claude Code Best Practices - Anthropic Engineering](https://www.anthropic.com/engineering/claude-code-best-practices)