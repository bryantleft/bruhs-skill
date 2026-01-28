---
description: Initialize bruhs config for existing projects
---

# init - Initialize Config

Set up `.claude/bruhs.json` for an existing project. Use this when you have a project that wasn't created with `/bruhs spawn`.

## Invocation

- `/bruhs init` - Initialize config for current project

## Prerequisites

- Existing project with git initialized
- In project root directory

## Workflow

### Step 1: Detect Existing Config

```bash
ls .claude/bruhs.json 2>/dev/null
```

If config exists:
```
Config already exists at .claude/bruhs.json
Would you like to reconfigure? [y/N]
```

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

If Linear is available:

```javascript
ToolSearch("select:mcp__linear__list_projects")

// Ask user to select team
"Which Linear team?"
teams.map(t => `○ ${t.name}`)

// Ask user to select project
projects = mcp__linear__list_projects({ teamId: selectedTeam.id })
"Which Linear project?"
projects.map(p => `○ ${p.name}`)
```

### Step 5: Confirm Stack Detection

Present detected stack and ask for confirmation/corrections:

```
Detected stack:
  ✓ Framework: Next.js
  ✓ Styling: Tailwind, shadcn
  ✓ Database: Drizzle + Postgres
  ✓ Testing: Vitest
  ✓ Tooling: Biome

Anything to add or change? [Enter to confirm]
```

Allow user to:
- Confirm detection
- Add missing items
- Correct wrong detections

### Step 6: Detect Installed Tooling

```bash
# Check user's Claude settings for installed MCPs and plugins
cat ~/.claude/settings.json 2>/dev/null
```

Extract:
- Installed MCP servers
- Enabled plugins

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
    "plugins": ["<detected-plugins>"]
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
  ✓ Plugins: superpowers, commit-commands

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
> /bruhs init

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

Confirm detected stack? [Y/n]
> Y

Creating config...
✓ Created .claude/bruhs.json

Ready! You can now use /bruhs cook and /bruhs yeet.
```
