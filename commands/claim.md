---
description: Claim an existing project for bruhs
---

# claim - Claim Existing Project

Set up `.claude/bruhs.json` for an existing project. Use this when you have a project that wasn't created with `/bruhs spawn`.

## Invocation

- `/bruhs claim` - Claim current project for bruhs

## Prerequisites

- Existing project with git initialized
- In project root directory

## Workflow

### Step 1: Detect Existing Config

```bash
ls .claude/bruhs.json 2>/dev/null
```

If config exists, use `AskUserQuestion`:

```javascript
AskUserQuestion({
  questions: [{
    question: "Config already exists at .claude/bruhs.json. Would you like to reconfigure?",
    header: "Reconfigure",
    multiSelect: false,
    options: [
      { label: "Yes", description: "Overwrite existing configuration" },
      { label: "No", description: "Keep existing configuration and exit" },
    ]
  }]
})

### Step 2: Detect Project Structure

```bash
# Check for monorepo indicators
ls turbo.json pnpm-workspace.yaml 2>/dev/null

# Check for framework indicators
ls next.config.* nuxt.config.* astro.config.* 2>/dev/null

# Check package.json for clues
cat package.json | jq '.dependencies, .devDependencies'
```

Auto-detect what's possible:
- **structure**: monorepo if turbo.json/pnpm-workspace.yaml exists
- **framework**: next.config.* → nextjs, etc.
- **styling**: tailwind.config.* → tailwind, components/ui → shadcn
- **database**: drizzle.config.* → drizzle, prisma/ → prisma
- **testing**: vitest.config.* → vitest, jest.config.* → jest
- **tooling**: biome.json → biome, .eslintrc → eslint

### Step 3: Check Linear MCP

```javascript
// Attempt to use Linear MCP
try {
  ToolSearch("select:mcp__linear__list_teams")
  teams = mcp__linear__list_teams()
  linearAvailable = true
} catch {
  console.log("Linear MCP not configured (optional).")
  linearAvailable = false
}
```

### Step 4: Gather Linear Config (if available)

If Linear is available, use `AskUserQuestion` for team and project selection:

```javascript
ToolSearch("select:mcp__linear__list_projects")

// Build team options dynamically
const teamOptions = teams.slice(0, 4).map(t => ({
  label: t.name,
  description: t.key || "Linear team"
}));

AskUserQuestion({
  questions: [{
    question: "Which Linear team?",
    header: "Team",
    multiSelect: false,
    options: teamOptions
  }]
})

// After team selected, get projects and ask
projects = mcp__linear__list_projects({ teamId: selectedTeam.id })

const projectOptions = projects.slice(0, 4).map(p => ({
  label: p.name,
  description: p.description || "Linear project"
}));

AskUserQuestion({
  questions: [{
    question: "Which Linear project?",
    header: "Project",
    multiSelect: false,
    options: projectOptions
  }]
})

### Step 5: Confirm Stack Detection

Present detected stack and use `AskUserQuestion` for confirmation:

First, output the detected stack:
```
Detected stack:
  ✓ Framework: Next.js
  ✓ Styling: Tailwind, shadcn
  ✓ Database: Drizzle + Postgres
  ✓ Testing: Vitest
  ✓ Tooling: Biome
```

Then ask for confirmation:

```javascript
AskUserQuestion({
  questions: [{
    question: "Is the detected stack correct?",
    header: "Confirm",
    multiSelect: false,
    options: [
      { label: "Yes, looks good", description: "Confirm and continue" },
      { label: "Add missing items", description: "I need to add something not detected" },
      { label: "Correct detection", description: "Something was detected incorrectly" },
    ]
  }]
})

### Step 6: Detect Installed Tooling

```bash
# Check user's Claude settings for installed MCPs
cat ~/.claude/settings.json 2>/dev/null
```

Extract:
- Installed MCP servers

### Step 6b: Detect Relevant Skills

Based on the detected stack, find skills that should be loaded for this project:

```javascript
// Always include workflow skills
detectedSkills = [
  "superpowers",       // brainstorming, debugging, verification, etc.
  "feature-dev",       // code-explorer, code-reviewer, code-architect
  "commit-commands",   // commit, commit-push-pr
]

// Add library/tech skills based on detected stack
skillsMap = {
  "shadcn": ["shadcn"],                          // styling includes shadcn
  "vercel-ai-sdk": ["vercel-ai-sdk"],            // ai: vercel-ai-sdk
  "better-auth": ["better-auth-best-practices"], // auth: better-auth
  "nextjs": ["vercel-react-best-practices"],     // framework: nextjs
}

if (stack.styling?.includes("shadcn")) detectedSkills.push("shadcn")
if (stack.ai === "vercel-ai-sdk") detectedSkills.push("vercel-ai-sdk")
if (stack.auth === "better-auth") detectedSkills.push("better-auth-best-practices")
if (stack.framework === "nextjs") detectedSkills.push("vercel-react-best-practices")

// Verify skills actually exist
Skill("find-skills")  // Use to validate detected skills
```

Output:
```
Detecting relevant skills...
✓ Found: shadcn (component library)
✓ Found: vercel-react-best-practices (React/Next.js patterns)
```

### Step 7: Create Config

Create `.claude/bruhs.json`:

```json
{
  "integrations": {
    "linear": {
      "team": "<selected-team>",
      "project": "<selected-project>",
      "labels": {
        "feat": "Feature",
        "fix": "Bug",
        "chore": "Chore",
        "refactor": "Improvement"
      }
    }
  },
  "tooling": {
    "mcps": ["<detected-mcps>"],
    "skills": ["<detected-skills>"]
  },
  "stack": {
    "structure": "<detected>",
    "framework": "<detected>",
    "styling": ["<detected>"],
    "database": ["<detected>"],
    "auth": null,
    "libraries": ["<detected>"],
    "state": "<detected>",
    "animation": null,
    "ai": "<detected>",
    "workers": null,
    "payments": null,
    "email": null,
    "testing": ["<detected>"],
    "tooling": ["<detected>"],
    "infra": ["<detected>"],
    "observability": [],
    "llmObservability": null
  }
}
```

If Linear not configured, omit the `linear` section from integrations.

### Step 8: Output Summary

```
Initialized .claude/bruhs.json

Integrations:
  ✓ Linear: Perdix Labs / Gambit

Stack detected:
  ✓ Framework: nextjs
  ✓ Styling: tailwind, shadcn
  ✓ Database: drizzle-postgres
  ✓ Testing: vitest
  ✓ Tooling: biome

Tooling:
  ✓ MCPs: linear, notion, context7
  ✓ Skills: superpowers, feature-dev, commit-commands, shadcn, vercel-react-best-practices

Ready! You can now use /bruhs cook and /bruhs yeet.
```

## Stack Detection Reference

| File/Pattern | Detected As |
|--------------|-------------|
| `next.config.*` | framework: nextjs |
| `nuxt.config.*` | framework: nuxt |
| `astro.config.*` | framework: astro |
| `tailwind.config.*` | styling: tailwind |
| `components/ui/` | styling: shadcn |
| `drizzle.config.*` | database: drizzle-postgres |
| `prisma/schema.prisma` | database: prisma |
| `convex/` | database: convex |
| `vitest.config.*` | testing: vitest |
| `jest.config.*` | testing: jest |
| `playwright.config.*` | testing: playwright |
| `biome.json` | tooling: biome |
| `.eslintrc*` | tooling: eslint |
| `.husky/` | tooling: husky |
| `zustand` in deps | state: zustand |
| `jotai` in deps | state: jotai |
| `@ai-sdk/*` in deps | ai: vercel-ai-sdk |
| `framer-motion` in deps | animation: framer-motion |
| `zod` in deps | libraries: zod |
| `@tanstack/react-query` in deps | libraries: tanstack-query |
| `effect` in deps | libraries: effect |
| `better-auth` in deps | auth: better-auth |
| `vercel.json` | infra: vercel |
| `Dockerfile` | infra: docker |
| `railway.json` | infra: railway |

## Example

```
> /bruhs claim

Checking for existing config...
✓ No existing config found

Detecting project structure...
✓ Monorepo: No (single package)
✓ Framework: Next.js (next.config.ts found)
✓ Styling: Tailwind, shadcn (tailwind.config.ts, components/ui/)
✓ Database: Drizzle + Postgres (drizzle.config.ts)
✓ Testing: Vitest (vitest.config.ts)
✓ Tooling: Biome, Husky (biome.json, .husky/)

Checking Linear MCP...
✓ Linear available

Which Linear team?
○ Perdix Labs ← selected

Which Linear project?
○ Gambit ← selected

Detecting relevant skills...
✓ Found: superpowers (workflow patterns)
✓ Found: feature-dev (code explorer/reviewer)
✓ Found: commit-commands (git workflows)
✓ Found: shadcn (component library)
✓ Found: vercel-react-best-practices (React/Next.js patterns)

Confirm detected stack? [Y/n]
> Y

Creating config...
✓ Created .claude/bruhs.json

Ready! You can now use /bruhs cook and /bruhs yeet.
```
