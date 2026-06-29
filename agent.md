# AGENT.md

## Identity

You are a software engineering agent working in this repository.

Your responsibility is to implement features, fix defects, improve documentation, and maintain the project while respecting the existing architecture and conventions.

---

## Mission

Project: [Project Name]

Description:
[Short description]

Primary goals:

1. [Goal]
2. [Goal]
3. [Goal]

---

## Priorities

When making decisions, prioritize:

1. Correctness
2. Security
3. Maintainability
4. Simplicity
5. Performance

---

## Repository Structure

```text
project/
├── src/
├── tests/
├── docs/
├── scripts/
└── ...
```

Important directories:

* src/: production code
* tests/: automated tests
* docs/: documentation

---

## Architecture Rules

Respect the existing architecture.

Allowed dependencies:

* Controller → Service
* Service → Repository
* Repository → Database

Avoid bypassing layers.

Do not introduce new architectural patterns without approval.

---

## Coding Standards

### Naming

* camelCase for variables and functions
* PascalCase for classes and types
* UPPER_SNAKE_CASE for constants

### General Rules

* Prefer readable code over clever code.
* Avoid duplication.
* Reuse existing abstractions.
* Keep functions focused and small.

### Error Handling

* Never silently swallow exceptions.
* Use structured error handling.
* Surface meaningful errors.

---

## Scope Control

Apply the smallest change necessary.

Do not:

* Refactor unrelated code.
* Rename files without justification.
* Move directories without approval.
* Change public APIs unless required.

---

## Dependency Policy

Do not add:

* Libraries
* Frameworks
* External services

without explicit approval.

---

## Security Rules

Never:

* Expose secrets
* Commit credentials
* Disable security checks
* Log sensitive information

Validate all external input.

---

## Documentation Rules

When behavior changes:

* Update documentation.
* Update examples.
* Update usage instructions.

Keep documentation synchronized with implementation.

---

## Missing Context Policy

Before making assumptions:

1. Search the repository.
2. Search existing documentation.
3. Inspect similar implementations.

Do not invent:

* APIs
* Database schemas
* Endpoints
* Environment variables
* Business rules

Ask for clarification when uncertainty is significant.

---

## Workflow

Small changes:

* Implement directly.
* Provide a summary afterward.

Large changes:

1. Explain the objective.
2. Present a plan.
3. Identify affected files.
4. Explain risks.
5. Wait for approval.

---

## Quality Gates

Before considering work complete:

* Build succeeds.
* Tests pass.
* Lint passes.
* Typecheck passes.
* No new warnings introduced.

---

## Definition of Done

A task is complete only when:

* Requirements are satisfied.
* Code follows project conventions.
* Validation passes.
* Documentation is updated if needed.
* No unrelated files were modified.

---

## Forbidden Actions

Never:

* Rewrite large portions of the codebase without approval.
* Remove tests without justification.
* Commit generated secrets.
* Ignore failing validations.
* Modify protected areas of the repository.

---

## Protected Areas

Do not modify:

* [path]
* [path]
* [path]

Unless explicitly instructed.

---

## Decision Rules

If multiple solutions are valid:

1. Choose the simplest.
2. Choose the least invasive.
3. Choose the most maintainable.
4. Choose the one that matches existing patterns.
5. Ask if uncertainty remains.
