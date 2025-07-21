# Claude Code Commands Context

## Key Concepts

- **Custom Command**: Specialized task definition in `.claude/commands/` that provides structured instructions for Claude to execute specific workflows
- **Command Anatomy**: Goal definition (what) + Context integration (how) + Validation/Evaluation (verification)
- **Argument Parsing**: Pattern `[TOPIC] [optional: file1 file2 ...]` where first part is main subject, additional parts are context files
- **Context Integration**: Commands rely on external playbook files (e.g., `@playbooks/playbook-creation.md`) for detailed "how" instructions
- **Validation vs Evaluation**: Objective verification (tests, compilation, syntax) vs subjective assessment (quality, engagement, effectiveness)
- **Clarifying Questions**: Required pre-execution questions to refine scope, audience, technical depth, and requirements
- **Delivery Format**: Structured output specification with clear formatting and validation requirements
- **Self-Assessment**: Claude's ability to verify its own work using predefined criteria from playbook files
- **Command Composition**: Ability to chain or combine commands for complex workflows

## Common Patterns

### Basic Command Structure
```markdown
# [Task Name] Task

You are an expert [domain specialist]. Your task is to [specific objective].

**Think deeply** about [key considerations] and approach this systematically.

## Your Objective
[Clear goal statement with context integration reference]

**Full Arguments**: $ARGUMENTS
Parse the arguments as: `[TOPIC] [optional: file1 file2 ...]`

## Context
Read [context-file.md] for precise instructions about [specific process].

## Required Clarifying Questions
1. **Scope Refinement**: [Question about boundaries]
2. **Technical Depth**: [Question about detail level]
3. **Authority Sources**: [Question about references]
4. **Validation Criteria**: [Question about success metrics]

## Delivery Format
Present the final [output type] with:
- [Specific requirement 1]
- [Specific requirement 2]
- [Validation method]
```
**When to use**: All custom commands follow this structure
**Expected output**: Clarifying questions followed by structured delivery

### Objective Validation Pattern
```markdown
## Validation Steps
1. **Syntax Check**: Run [tool/command] to verify correctness
2. **Functional Test**: Execute [test command] to confirm operation
3. **Output Verification**: Check that [specific criteria] are met
4. **Integration Test**: Verify compatibility with [system/context]

**Success Criteria**:
- [Measurable outcome 1]
- [Measurable outcome 2]
- [Error-free execution confirmation]
```
**When to use**: Commands producing code, configurations, or testable outputs
**Expected output**: Pass/fail validation with specific error reporting

### Subjective Evaluation Pattern
```markdown
## Evaluation Framework
### Quality Assessment
- **[Criterion 1]**: [Specific evaluation question]
- **[Criterion 2]**: [Specific evaluation question]
- **[Criterion 3]**: [Specific evaluation question]

### Scoring Method
Rate each criterion 1-5:
- 5: Excellent - [specific description]
- 4: Good - [specific description]
- 3: Acceptable - [specific description]
- 2: Needs improvement - [specific description]
- 1: Poor - [specific description]

### Improvement Recommendations
For scores 4 or below, provide:
- **Issue**: [What needs improvement]
- **Impact**: [Why this matters]
- **Recommendation**: [Specific actionable fix]
```
**When to use**: Commands producing content, designs, or qualitative outputs
**Expected output**: Structured evaluation with improvement recommendations

## Implementation Details

### Creating a Custom Command

1. **Create Command File**
```bash
touch .claude/commands/[command-name].md
```

2. **Define Command Structure**
```markdown
# [Task Name] Task

You are an expert [domain]. Your task is to [objective].

**Think deeply** about [considerations] and approach this systematically.

## Your Objective
[Goal with context reference]

**Full Arguments**: $ARGUMENTS
```

3. **Add Argument Parsing**
```markdown
Parse the arguments as: `[MAIN_TOPIC] [optional: file1 file2 ...]`

The first part should be the [main subject].
Any `filename` references should be loaded as complementary context files.
```

4. **Include Context Integration**
```markdown
## Context
Read [context-file.md] for precise instructions about [process].
```

5. **Define Clarifying Questions**
```markdown
## Required Clarifying Questions
1. **Scope**: [Boundary question]
2. **Depth**: [Detail level question]
3. **Sources**: [Authority/reference question]
4. **Validation**: [Success criteria question]
```

6. **Specify Delivery Format**
```markdown
## Delivery Format
Present the final [output] with:
- [Requirement 1]
- [Requirement 2]
- [Validation method]
```

**Validation**: Command file should parse correctly and reference existing context files

### Context File Integration

1. **Reference Pattern**
```markdown
## Context
Read @playbooks/[playbook-name].md for [specific instructions].
```

2. **Multiple Context Files**
```markdown
## Context
- @playbooks/[primary-playbook].md - [primary instructions]
- @playbooks/[secondary-playbook].md - [supplementary guidance]
```

3. **Argument-Based Context Loading**
```markdown
Any `@filename` references in arguments will be automatically loaded as complementary context files.
```

**Validation**: All referenced context files must exist and provide complete guidance

### Validation Implementation

1. **Objective Validation Setup**
```markdown
## Validation Steps
1. Run `[command]` to check [specific aspect]
2. Verify output contains [expected elements]
3. Confirm [measurable criteria] are met

**Success Criteria**: [Pass/fail conditions]
```

2. **Subjective Evaluation Setup**
```markdown
## Evaluation Criteria
Use the following framework from [context-file]:
- [Criterion 1]: [Assessment method]
- [Criterion 2]: [Assessment method]

**Scoring**: Rate 1-5 with specific improvement recommendations for scores <4
```

**Validation**: Evaluation criteria must be specific enough for consistent assessment

## Validation Methods

### Command File Validation
- **Syntax Check**: Markdown formatting is correct
- **Reference Check**: All `@context` references point to existing files
- **Structure Check**: Contains all required sections (Objective, Context, Questions, Delivery)
- **Argument Pattern**: Follows `$ARGUMENTS` parsing specification

### Context Integration Validation
- **Completeness**: Referenced context files provide sufficient guidance
- **Consistency**: Command objectives align with context capabilities
- **Dependencies**: All required external resources are available

### Execution Validation
- **Clarifying Questions**: All required questions are asked before execution
- **Output Structure**: Delivery follows specified format requirements
- **Quality Check**: Output meets validation/evaluation criteria defined in command

## Troubleshooting

### Validation Failures
**Symptoms**: Output doesn't meet specified criteria
**Cause**: Unclear or missing validation specifications in command
**Fix**: Add specific, measurable validation steps with clear pass/fail criteria

### Context Integration Issues
**Symptoms**: Command asks for information available in context files
**Cause**: Context files don't provide actionable guidance for command objectives
**Fix**: Update context files to include specific patterns and examples for command use case

### Argument Parsing Problems
**Symptoms**: Command doesn't properly handle multiple arguments
**Cause**: Argument pattern specification is ambiguous
**Fix**: Use exact pattern `[TOPIC] [optional: file1 file2 ...]` with clear parsing instructions

## Authoritative References

- [Claude Code Commands Documentation](https://docs.anthropic.com/en/docs/claude-code/slash-commands)
- [Context Creation Framework](contexts/context-creation.md)
- [Existing Command Examples](.claude/commands/)
- [Claude Code Settings](https://docs.anthropic.com/en/docs/claude-code/settings)

## Quality Checklist

- [ ] Command file follows required structure with all sections
- [ ] Argument parsing pattern is clearly specified
- [ ] All context file references exist and provide complete guidance
- [ ] Clarifying questions cover scope, depth, sources, and validation
- [ ] Validation/evaluation criteria are specific and measurable
- [ ] Delivery format requirements are actionable
- [ ] Troubleshooting covers common failure modes

## Validation Test

**Prompt**: "Using only the information in this context module, create a custom Claude Code command that generates technical documentation with both objective validation (syntax checking) and subjective evaluation (content quality assessment)."

**Success Criteria**: AI agent creates a complete command file with:
1. Proper structure and argument parsing
2. Context file integration
3. Specific clarifying questions
4. Both objective and subjective validation methods
5. Clear delivery format specification