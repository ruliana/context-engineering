# Create Playbook Task

You are an expert technical documentation specialist and AI-optimized content creator. Your task is to create authoritative, self-contained playbooks that provide AI agents with domain-specific knowledge and actionable information.

**Think deeply** about the subject matter and approach this systematically.

## Your Objective

Create a comprehensive playbook for the specified domain/technology following the playbook creation framework in @playbooks/playbook-creation.md. The playbook must be optimized for AI agent consumption and provide complete knowledge without external dependencies.

**Full Arguments**: $ARGUMENTS

Parse the arguments as: `[TOPIC] [optional: file1 file2 ...]`

The first part should be the main topic for the playbook. Any `filename` references in the arguments should be loaded as complementary playbook files.

## Context

Read @playbooks/playbook-creation.md for precise instructions about how to write playbooks.

## Required Clarifying Questions

Before creating the playbook, ask these essential questions about the specified topic:

1. **Scope Refinement**: What specific aspects of the topic should be covered? (e.g., basic usage, advanced configuration, specific use cases)
2. **Use Case Focus**: What are the primary use cases or scenarios this playbook should cover? (e.g., basic setup, production deployment, troubleshooting, integration patterns)
3. **Technical Depth**: What level of technical detail is needed? (e.g., beginner setup, advanced configuration, enterprise patterns)
4. **Authority Sources**: Are there specific official documentation sources or authoritative references you want prioritized?
5. **Environment Context**: Are there specific environments, versions, or constraints to consider? (e.g., Linux/Windows, specific versions, cloud platforms)

## Delivery Format

Present the final playbook as a complete markdown document following the framework structure in @playbooks/playbook-creation.md. Ensure it's immediately usable by AI agents without modification.

**Important Requirements:**
- **Naming**: Name the playbook after the topic with "-playbook" suffix (e.g., `postgresql-administration-playbook.md`, `docker-containerization-playbook.md`, `fastapi-development-playbook.md`). Avoid action verbs like "create", "improve", "setup"
- **Location**: Place all playbooks in the `playbooks/` folder
- **Format**: Use `.md` extension and follow markdown best practices
- **Structure**: Follow the template and guidelines provided in the complementary playbook files above

## Validation Steps

1. **Structure Check**: Verify the playbook contains all required sections from the framework
2. **Reference Validation**: Confirm all authoritative references exist and are accessible
3. **Completeness Assessment**: Ensure all key concepts, patterns, and implementation details are covered
4. **AI-Agent Optimization**: Validate content is structured for AI consumption with clear patterns and examples
5. **Self-Containment Test**: Verify the playbook provides complete knowledge without external dependencies

**Success Criteria**:
- Playbook follows the exact structure from @playbooks/playbook-creation.md
- All referenced sources are authoritative and current
- Contains actionable patterns and implementation details
- Provides complete domain coverage for the specified topic
- Can be consumed by AI agents without additional context
