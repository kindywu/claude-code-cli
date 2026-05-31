# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Tech Stack

- **Runtime**: Bun (not Node.js)
- **Language**: TypeScript/TSX (strict mode)
- **UI**: Ink (React for the terminal) — the custom fork lives in `ink/`
- **CLI framework**: Commander.js (`@commander-js/extra-typings`)
- **Linting**: Biome (imports) + ESLint (custom rules, e.g. `custom-rules/no-top-level-side-effects`)

## Architecture Overview

This is the **Claude Code CLI** source — a terminal-based AI coding assistant. The architecture follows a hub-and-spoke pattern where the core engine orchestrates tools, commands, skills, and background tasks.

### Entry Points (`entrypoints/`)

- `cli.tsx` — Bootstrap entry. Handles fast-path flags (`--version`, `--help`) before loading the full CLI. Sets env vars (corepack pinning, CCR heap size, ablation baseline).
- `init.ts` — Post-bootstrap initialization (telemetry, GrowthBook, policy limits).
- `agentSdkTypes.ts` — SDK-level types shared between the CLI and programmatic SDK surface (`entrypoints/sdk/`).
- `mcp.ts` — Standalone MCP server startup entrypoint.

### Core Engine

- **`main.tsx`** (800KB) — The central orchestrator. Defines the Commander.js CLI structure, all flags/options, and the REPL launch logic. Imports are ordered carefully: side-effect prefetches (MDM, keychain) run before module evaluation.
- **`QueryEngine.ts`** — The AI conversation loop. Feeds messages to the model, processes tool use responses, handles compaction boundaries, structured output enforcement, and file history snapshots. Delegates actual API calls to `query.ts`.
- **`query.ts`** (68KB) — Low-level model query logic. Constructs API requests (system prompt, messages, tools, thinking budget), handles streaming, retries, and usage tracking.
- **`setup.ts`** — Session initialization. Sets CWD, project root, git worktree creation, session memory init, file watcher startup.

### Command System (`commands.ts` + `commands/`)

Each slash command (`/commit`, `/review`, `/mcp`, etc.) is a module in `commands/<name>/`. Commands are registered via `getCommands()` in `commands.ts`. Three command types exist:
- **prompt** — Injects text into the conversation
- **local** — Executes logic locally without an API round-trip
- **dialog** — Launches an interactive dialog/modal

Some commands (`commit.ts`, `review.ts`, `init.ts`) are single files; others are directories with their own components and logic.

### Tool System (`tools.ts` + `Tool.ts` + `tools/`)

Tools are callable functions exposed to the AI model (Bash, FileRead, FileEdit, Grep, etc.). Each tool lives in `tools/<ToolName>/` and conforms to the `Tool` interface defined in `Tool.ts`:
- JSON Schema input validation
- `prompt()` — returns the tool definition sent to the model
- `isEnabled()` / `isReadOnly()` — permission gating
- `call()` — execution logic

**Feature-gated tools** use `require()` inside `feature()` checks for dead code elimination in external builds. Internal-only tools (REPL, SuggestBackgroundPR) are gated behind `USER_TYPE === 'ant'`.

### Background Tasks (`tasks.ts` + `Task.ts` + `tasks/`)

Long-running operations (shell commands, sub-agents, remote agents) run as background tasks with lifecycle tracking. Task types: `local_bash`, `local_agent`, `remote_agent`, `in_process_teammate`, `local_workflow`, `monitor_mcp`, `dream`. Tasks transition through statuses: `pending` → `running` → `completed`/`failed`/`killed`.

### Services (`services/`)

- **`services/api/`** — Anthropic API communication (bootstrap, files, Claude models, logging)
- **`services/mcp/`** — Full MCP (Model Context Protocol) implementation: connection management, OAuth, SSE/stdio transports, tool/resource proxying
- **`services/analytics/`** — GrowthBook feature flags, event logging
- **`services/lsp/`** — Language Server Protocol integration
- **`services/settingsSync/`** — Cross-machine settings synchronization
- **`services/remoteManagedSettings/`** — MDM/enterprise policy management

### UI Components (`components/`)

Ink-based React components for the terminal UI. Structure:
- `components/` — Flat component files (App.tsx, Messages.tsx, Message.tsx, etc.)
- `components/design-system/` — Reusable design system primitives
- `components/agents/`, `components/mcp/`, `components/memory/`, `components/skills/`, `components/tasks/`, `components/teams/` — Feature-specific UI clusters
- `components/messages/` — Message rendering components
- `components/permissions/`, `components/sandbox/` — Permission/sandbox UI

### Ink Framework Fork (`ink/`)

A substantially customized fork of Ink (React terminal renderer). Includes custom: reconciler, layout engine, text measurement, ANSI handling, input/keypress parsing, terminal focus/blur, hyperlink support. This is not the npm `ink` package — it's a local fork with deep modifications.

### Context System (`context.ts`)

Builds the system and user context injected into every API request. Sources include: CLAUDE.md files, git status, git branch info, memory files, system prompt injection, and platform/date metadata. Results are memoized.

### Skills & Plugins

- **`skills/`** — The skills framework. Bundled skills in `skills/bundled/`. Skill invocation creates a sub-agent with specialized instructions.
- **`plugins/`** — Plugin system for extensibility. Built-in plugins in `plugins/builtinPlugins.ts`. Plugins can contribute commands, tools, hooks, and skills.

### Agent/Swarm System

- **`tools/AgentTool/`** — Spawns sub-agents (Explore, Plan, general-purpose, etc.) for parallel or isolated work
- **`components/agents/`** — Agent status UI
- **`utils/agentSwarmsEnabled.ts`** — Feature gate for multi-agent swarms
- Agent isolation modes include git worktree isolation

### State Management (`state/`)

Application state managed via `AppState.tsx` / `AppStateStore.ts` using a React context + reducer pattern (not Zustand, despite naming similarities). `selectors.ts` provides memoized state selectors.

### Key Utilities (`utils/`)

33+ utility modules covering: auth, git, config, permissions, hooks (user-configured hooks), file watching, clipboard, worktree management, session storage, startup profiling, and more.

## Important Patterns

### Feature Flagging via `bun:bundle`

```typescript
import { feature } from 'bun:bundle'

const someModule = feature('FEATURE_NAME')
  ? require('./heavy-module.js').thing
  : null
```

The `feature()` function is a Bun macro that resolves at build time. External/public builds have most features stripped, resulting in smaller binaries. Internal (Anthropic) builds include all features. Always use `require()` (not `import`) inside feature gates so dead code elimination works.

### Internal-only Code (ANT)

Code gated behind `process.env.USER_TYPE === 'ant'` or `feature('...')` is for Anthropic internal use only. This includes the REPL tool, SuggestBackgroundPR, Kairos/Proactive features, Bridge mode, Voice mode, and the agents platform.

### Custom ESLint Rules

The project uses custom ESLint rules prefixed `custom-rules/`:
- `custom-rules/no-top-level-side-effects` — Prohibits side effects at module scope (enforced on most files except `main.tsx` and `entrypoints/cli.tsx` which need them for startup optimization)
- `custom-rules/no-process-env-top-level` — Prevents reading `process.env` at import time
- `custom-rules/safe-env-boolean-check` — Enforces safe boolean checks on env vars

### Import Ordering

`main.tsx` and a few other core files disable import organization (`biome-ignore-all assist/source/organizeImports`) with the marker `ANT-ONLY import markers must not be reordered`. This is because side-effect ordering matters for startup performance — MDM prefetch, keychain reads, and profile checkpoints must run before other modules load.

### Circular Dependencies

Some circular dependencies are broken via lazy `require()` inside getter functions rather than top-level imports (see `tools.ts` for TeamCreateTool, TeamDeleteTool, SendMessageTool).

## Key Constants (`constants/`)

- `oauth.ts` — OAuth configuration
- `product.ts` — Product URLs, API endpoints
- `xml.ts` — XML tag constants for structured output
- `system.ts` — System prompt configuration
- `tools.ts` — Tool name constants
- `messages.ts` — Message-related constants
- `prompts.ts` — Prompt templates
