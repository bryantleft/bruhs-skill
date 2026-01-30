---
name: bruhs
description: Opinionated development lifecycle - spawn projects, cook features, yeet to ship
---

# bruhs - Complete Development Lifecycle

When invoked, use the `AskUserQuestion` tool to present the command options interactively:

```javascript
AskUserQuestion({
  questions: [{
    question: "What do you want to do?",
    header: "Command",
    multiSelect: false,
    options: [
      { label: "spawn", description: "Create new project or add to monorepo" },
      { label: "claim", description: "Claim existing project for bruhs" },
      { label: "cook", description: "Plan + Build a feature end-to-end" },
      { label: "yeet", description: "Ship: Linear ticket → Branch → Commit → PR" },
      { label: "peep", description: "Address PR review comments and merge" },
      { label: "dip", description: "Clean up after merge and switch to base branch" },
    ]
  }]
})
```

**Note:** The `slop` command is intentionally excluded from the main menu as it's a specialized cleanup tool. Users can invoke it directly with `/bruhs slop`.

After user selects, follow the corresponding command file:
- **spawn** → Read and follow `commands/spawn.md`
- **claim** → Read and follow `commands/claim.md`
- **cook** → Read and follow `commands/cook.md`
- **yeet** → Read and follow `commands/yeet.md`
- **peep** → Read and follow `commands/peep.md`
- **dip** → Read and follow `commands/dip.md`
- **slop** → Read and follow `commands/slop.md`

## Quick Access

Users can also specify directly:
- `/bruhs spawn` or `/bruhs spawn <name>`
- `/bruhs claim`
- `/bruhs cook` or `/bruhs cook <feature>`
- `/bruhs yeet`
- `/bruhs peep` or `/bruhs peep <PR#>` or `/bruhs peep <TICKET-ID>`
- `/bruhs dip`
- `/bruhs slop` or `/bruhs slop <path>` or `/bruhs slop --fix`

If an argument is provided, skip the selection and go directly to that command.
