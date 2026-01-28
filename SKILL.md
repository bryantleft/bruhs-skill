---
name: bruhs
description: Opinionated development lifecycle - spawn projects, cook features, yeet to ship
---

# bruhs - Complete Development Lifecycle

A plugin for the complete development lifecycle: project creation, feature development, and shipping.

## Commands

| Command | Purpose |
|---------|---------|
| `/bruhs:spawn` | Create new project or add to monorepo |
| `/bruhs:cook` | Plan + Build a feature end-to-end |
| `/bruhs:yeet` | Ship: Linear ticket → Branch → Commit → PR |

## Workflow

```
/bruhs:spawn → Create project/app with full stack setup
    ↓
/bruhs:cook → Plan and build features with TDD
    ↓
/bruhs:yeet → Ship to Linear + GitHub
```

## Configuration

Each project should have a `.claude/bruhs.json` file:

```json
{
  "linear": {
    "team": "Your Team",
    "project": "Your Project",
    "labelMapping": {
      "feat": "Feature",
      "fix": "Bug",
      "chore": "Chore",
      "refactor": "Improvement"
    }
  },
  "stack": {
    "structure": "turborepo",
    "framework": "nextjs",
    "styling": ["tailwind", "shadcn"],
    "database": ["drizzle-postgres"],
    "auth": "better-auth"
  }
}
```

## Core Principles

| Principle | Description |
|-----------|-------------|
| **Atomic Design** | Hierarchical component architecture |
| **Clean** | Simple, readable, maintainable code |
| **Immutability** | Predictable state and data flow |
| **Scalable** | Architecture that grows with your needs |
| **Maintainable** | Long-term sustainability and extensibility |
| **Single Source of Truth** | One authoritative source for each piece of data |

## Dependencies

### Linear MCP (optional)

If Linear MCP is not configured, skills offer git-only mode.

### GitHub CLI

Uses `gh` for PR creation. Authenticate with `gh auth login`.
