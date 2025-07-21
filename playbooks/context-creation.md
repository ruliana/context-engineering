# Context Creation Framework

Framework for developing authoritative, AI-agent-optimized context modules that provide deep domain knowledge and composable expertise.

## Core Objective

Create self-contained context modules that AI agents can consume to gain domain expertise. Context modules are designed to be composable - they can be combined for complex, multi-domain tasks. Focus: actionable knowledge over theoretical concepts.

## Authority Identification

### Source Priority Order
1. **Official Documentation**: postgres.org, python.org, anthropic.com, openai.com
2. **Primary Sources**: Original research, RFCs, specifications
3. **Expert Publications**: Industry leaders, .gov/.edu sources
4. **Peer-Reviewed**: Academic papers with citations

### Domain Authority Map
- **Technology**: Official docs → GitHub → Stack Overflow (accepted answers)
- **AI/ML**: anthropic.com → openai.com → arxiv.org → research institutions
- **Cloud**: aws.amazon.com → cloud.google.com → azure.microsoft.com
- **Databases**: Official project sites → documentation → community wikis
- **Languages**: Official language sites → standard libraries → core team blogs

### Quick Validation
- Check publication date (prefer recent)
- Verify author credentials
- Confirm primary source citations
- Test examples/code snippets work

## AI-Agent Optimized Structure

### Required Sections
```markdown
# [Topic] Context

## Key Concepts
[Essential domain knowledge - definitions, core principles]

## Common Patterns
[Typical approaches, standard configurations, decision trees]

## Implementation Details
[Code examples, commands, specific procedures]

## Validation Methods
[How to verify success, common error patterns]

## Authoritative References
[Official sources only - no tutorials or blogs]
```

### Writing for AI Consumption

**Direct Information Delivery**:
- Lead with facts, not context
- Use bullet points over paragraphs
- Include exact commands/code
- Provide decision criteria

**Pattern Recognition**:
- Show multiple examples of same pattern
- Highlight variations and when to use each
- Include both success and failure examples

**Self-Contained Context**:
- Define all domain terms upfront
- No references to external tutorials
- Complete examples that work independently

## Good vs Bad Examples

### ❌ Bad: Too Abstract
```markdown
## Database Connection
When working with databases, it's important to consider various factors 
such as connection pooling, security, and performance optimization. 
There are many approaches to database connectivity...
```

### ✅ Good: Actionable and Specific
```markdown
## Database Connection Patterns

### PostgreSQL Connection (Python)
```python
import psycopg2
conn = psycopg2.connect(
    host="localhost",
    database="mydb", 
    user="username",
    password="password"
)
```

### Connection Pool Pattern
```python
from psycopg2 import pool
connection_pool = psycopg2.pool.SimpleConnectionPool(1, 20, **conn_params)
```

### Validation
- Test: `SELECT 1` should return single row
- Error: `psycopg2.OperationalError` indicates connection failure
```

### ❌ Bad: Human-Focused Explanation
```markdown
## Getting Started with Docker
Docker is a containerization platform that has revolutionized how we deploy applications. 
In this section, we'll explore the fundamental concepts that every developer should understand 
before diving into container orchestration...
```

### ✅ Good: AI-Agent Direct
```markdown
## Docker Core Commands

### Image Management
- `docker build -t name:tag .` - Build from Dockerfile
- `docker pull image:tag` - Download image
- `docker images` - List local images

### Container Operations  
- `docker run -d -p 8080:80 nginx` - Run detached with port mapping
- `docker ps` - List running containers
- `docker logs container_id` - View logs
- `docker exec -it container_id bash` - Interactive shell

### Validation Patterns
- Container running: `docker ps` shows STATUS "Up"
- Port accessible: `curl localhost:8080` returns response
- Logs available: `docker logs` shows application output
```

### ❌ Bad: Vague Best Practices
```markdown
## Security Considerations
Security should always be a top priority when developing applications. 
Consider implementing proper authentication, use HTTPS, validate inputs, 
and follow security best practices for your specific technology stack.
```

### ✅ Good: Specific Security Patterns
```markdown
## Security Implementation Patterns

### Input Validation (Python)
```python
import re
def validate_email(email):
    pattern = r'^[a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,}$'
    return re.match(pattern, email) is not None
```

### Environment Variables
```bash
# Set secrets via environment
export DATABASE_URL="postgresql://user:pass@localhost/db"
export API_KEY="secret_key_here"
```

### HTTPS Enforcement (nginx)
```nginx
server {
    listen 80;
    return 301 https://$server_name$request_uri;
}
```

### Validation Checks
- Environment variables: `echo $DATABASE_URL` shows value
- HTTPS redirect: `curl -I http://domain.com` returns 301
- Input validation: Invalid emails return False
```

## Creation Process

### 1. Identify Official Sources
- Find primary documentation site
- Locate official API references  
- Check for RFCs or specifications
- Verify source authority and currency

### 2. Extract Core Patterns
- Document common usage patterns
- Include exact syntax/commands
- Show configuration examples
- Note version-specific differences

### 3. Test All Examples
- Verify every code snippet runs
- Test commands in clean environment
- Document expected outputs
- Include error scenarios

### 4. Validate with AI Agent
- Load context module in clean conversation
- Ask agent to perform related tasks
- Verify agent can act without additional context
- Refine based on AI comprehension gaps

## Context Module Template

```markdown
# [Technology/Tool] Context

## Key Concepts
- [Term]: [Definition]
- [Term]: [Definition]
- [Term]: [Definition]

## Common Patterns

### [Pattern Name]
```[language]
[exact code example]
```
**When to use**: [specific criteria]
**Expected output**: [what success looks like]

### [Pattern Name]
```[language]
[exact code example]
```
**When to use**: [specific criteria]
**Expected output**: [what success looks like]

## Implementation Details

### [Specific Task]
1. [Exact command/code]
2. [Exact command/code]
3. [Exact command/code]

**Validation**: [How to verify it worked]
**Common errors**: [Error message] = [solution]

## Troubleshooting

### [Error Pattern]
**Symptoms**: [What you see]
**Cause**: [Why it happens]
**Fix**: [Exact solution]

### [Error Pattern]
**Symptoms**: [What you see]
**Cause**: [Why it happens] 
**Fix**: [Exact solution]

## Authoritative References
- [Official Documentation](url)
- [API Reference](url)
- [Specification](url)
```

## Quality Checklist

- [ ] All code examples tested and working
- [ ] Every command includes expected output
- [ ] Error messages documented with solutions
- [ ] No external tutorial dependencies
- [ ] Validated by running with AI agent
- [ ] Sources are primary/official only
- [ ] Patterns show multiple working examples

## Validation Test

**Prompt**: "Using only the information in this context module, perform [specific task related to the domain]"

**Success criteria**: AI agent completes task without asking for additional information or external resources.