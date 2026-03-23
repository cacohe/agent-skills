# Coding Skills Specification

## Project Overview

- **Project Name**: Coding Skills
- **Type**: Agent Skills Library
- **Core Functionality**: A collection of coding skills and practices for AI agents, including safety guardrails and code quality standards.
- **Target Users**: AI coding agents and developers

## Modules

### 1. Safe Coding (`safe-coding/`)
Agent safety guardrails for operations involving:
- File system operations
- Shell commands and processes
- Network and API calls
- Database operations
- Secrets and sensitive information
- Permissions and infrastructure

### 2. Write Code (`write-code/`)
Commercial-grade code writing standards covering:
- Project directory structure
- Clean Code naming conventions
- Function design principles
- Documentation standards
- Logging module (independent封装)
- Extensibility design
- Reusability design

## Directory Structure

```
.
├── README.md
├── SPEC.md
├── .gitignore
└── coding-skills/
    ├── safe-coding/
    │   ├── SKILL.md
    │   └── references/
    │       ├── dangerous-patterns.md
    │       └── safe-alternatives.md
    └── write-code/
        └── SKILL.md
```

## Usage

Load the appropriate skill based on the task:
- Use `safe-coding` for safety-related operations
- Use `write-code` when generating commercial-grade code
