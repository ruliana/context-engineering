# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is the AI Playbooks repository - a systematic framework for creating structured knowledge modules that provide AI agents with domain-specific expertise for complex tasks. The repository contains:

- **Playbooks**: Self-contained knowledge modules optimized for AI consumption
- **Examples**: Practical demonstrations of multi-playbook composition  
- **Framework**: Systematic approach for building authoritative playbooks

## Architecture

### Core Concept
Playbooks function as "subjects" that can be paired with any "verb" (action), similar to REST APIs. The playbook provides domain knowledge while instructions specify actions to perform.

### Repository Structure
```
playbooks/         # All knowledge modules (meta and domain-specific)
├── playbook-creation.md           # Meta framework for building playbooks
├── [topic]-playbook.md           # Domain-specific playbooks
examples/          # Practical demonstrations of multi-playbook usage
├── claude-code-noninteractive-blog/  # Blog post using multiple playbooks
```

### Playbook Framework
Every playbook follows this consistent structure:
- **Key Concepts**: Essential domain knowledge and definitions
- **Common Patterns**: Standard approaches and decision frameworks
- **Implementation Details**: Executable code examples and procedures
- **Validation Methods**: Success criteria and troubleshooting
- **Authoritative References**: Official documentation only

## Development Workflow

### No Build/Test Commands
This repository contains only Markdown documentation files. There are no build systems, package managers, or test frameworks to run.

### Content Validation
- All code examples in playbooks must be tested independently
- Examples should work in clean environments without dependencies
- Focus on official documentation and primary sources only

### Creating New Playbooks
1. Follow the framework in `playbooks/playbook-creation.md`
2. Use only authoritative sources (official docs, RFCs, specifications)
3. Structure content for AI consumption with direct, actionable information
4. Test all code examples work independently
5. Validate with AI agents before finalizing

### Playbook Composition
Playbooks are designed to be composable - multiple playbooks can be combined for complex, multi-domain tasks using the `@playbook-name.md` syntax.

## Key Principles

- **Authority-First Sources**: Official documentation and primary sources only
- **AI-Agent Optimization**: Structured for immediate AI consumption and application  
- **Self-Contained**: Complete domain knowledge without external dependencies
- **Actionable**: Executable commands and code that work independently
- **Composable**: Designed to work independently or combined with other playbooks

## License

Creative Commons Attribution 4.0 International License (CC BY 4.0)