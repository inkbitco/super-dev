---
inclusion: auto
fileMatchPattern: "src/**/*.ts, prompts/*.md"
description: Checklist for adding or modifying MCP tools, prompts, and feature groups
---

# Adding Tools, Prompts, or Features

When adding or modifying tools, prompts, or feature groups, update ALL of the following locations. Missing any of these will cause inconsistencies between the code, docs, and configuration.

## Checklist: New Tool

1. **Implementation** — create or edit a file in `src/lib/`, export a `ToolDef[]` array
2. **Registration** — import and register in `src/index.ts` in the appropriate loop (project-root, standalone, or upstream)
3. **Feature group** — add the tool name to `TOOL_GROUPS` in `src/index.ts` so it respects `SUPER_DEV_DISABLE`
4. **README Quick Reference** — add to the Tools section in `README.md`
5. **README Features** — add or update the extended description in the Features section of `README.md`
6. **README Disabling Features** — update the feature group table if a new group is introduced

## Checklist: New Prompt

1. **Prompt file** — create a `.md` file in `prompts/`. The first `# Heading` becomes the command description.
2. **Feature group** — add the prompt name (filename without `.md`) to `PROMPT_GROUPS` in `src/index.ts`
3. **README Quick Reference** — add to the Slash Commands table in `README.md`
4. **README Features** — add or update the extended description in the Features section of `README.md`
5. **README Disabling Features** — update the feature group table if a new group is introduced

## Checklist: New Feature Group

1. **TOOL_GROUPS / PROMPT_GROUPS** — add mappings in `src/index.ts`
2. **README Disabling Features** — add the group to the table in `README.md`
3. **README Features** — add a new subsection with extended description

## Tool Definition Shape

```ts
import { z } from "zod";
import { ok, err } from "../types.js";
import type { ToolDef, AppContext } from "../types.js";

const mySchema = {
  name: z.string().describe("Human-readable description for the agent"),
  verbose: z.boolean().optional(),
};

async function myHandler(
  args: Record<string, unknown>,
  { projectRoot }: AppContext
): Promise<ToolResult> {
  const name = args.name as string;
  return ok(`Result: ${name}`);
}

export const myTools: ToolDef[] = [
  {
    name: "my_tool",
    description: "What this tool does — shown to the AI agent.",
    schema: mySchema,
    handler: myHandler,
  },
];
```

## Registration Categories (in index.ts)

Tools are registered in three groups based on their needs:

1. **Project-root tools** (spec, rules) — `await resolveProjectRoot()` before handler
2. **Standalone tools** (thread history, TTS) — no project root needed, skip resolution
3. **Upstream tools** — project root + dynamic enable/disable for merge-only tools

## Conventions

- Schema values are bare Zod types, NOT wrapped in `z.object()`
- Handler args are `Record<string, unknown>` — cast individual fields with `as`
- Always return `ToolResult` via `ok()` or `err()` — never construct the shape manually
- Tool names use `snake_case` (e.g., `spec_create`, `upstream_status`)
- Prompt filenames use `kebab-case` (e.g., `spec-plan.md`, `design-review.md`)
- Descriptions should be concise but include enough context for the AI agent to know when to use the tool
