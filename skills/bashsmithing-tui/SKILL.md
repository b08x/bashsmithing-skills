---
name: bashsmithing-tui
description: Terminal UI scaffolder and advisor for Bash projects using the Charm/Gum ecosystem. Activates on any mention of: TUI, terminal UI, terminal interface, terminal app, Gum, gum prompts, interactive menu, text input, multi-line input, spinner, confirm prompt, filter list, table rendering, markdown pager, keyboard navigation, cursor movement, human in the loop component, HIL interface, RAG viewer, agent control panel, streaming output panel, text input, spinner, list selection, metrics display, or progress bar within a Bash script context. Always runs bashsmithing-context as prerequisite for gum CLI verification. Always produces full skeleton: bin/ entry point, lib/gum.sh wrapper, lib/menu.sh, and settings/config.sh.
---

# Bashsmithing — TUI

Scaffolder and advisor for terminal UI applications using the Bash-native Charm/Gum ecosystem.
Always produces a full skeleton. Generates against verified gum CLI syntax only.

## Step 1: Prerequisites (Always Run First)

Before generating any scaffold or component code:

1. **Activate bashsmithing-context** for gum CLI verification.
   Confirm current gum version and available flags. Generate no code until CLI syntax is confirmed or WARNING block is injected.

2. **Extract domain** from the request — what does this TUI control or display?

3. **Identify menus** — what interactive paths does the user need?

4. **Identify inputs per screen** — single-line, multi-line, selections, filters.

5. **Identify long-running tasks** — where are spinners needed?

6. **Identify external data flows** — what CLI tools or APIs does this TUI interact with?

**Compound prompts** (e.g., "refactor this data pipeline AND build a TUI for it"):
Handle the TUI component here. State explicitly:
"Handling the TUI dashboard component. The data pipeline component should be
addressed with bashsmithing-refactor."

## Step 2: Detect Mode

**Scaffolding** — triggered by: create, build, scaffold, generate, write a TUI for.
Output: full skeleton (see structure below) + complete file content.

**Advisory** — triggered by: how do I, which gum command, explain, what's the best way.
Output: recommendation + minimal snippet. No full scaffold unless asked.

## Skeleton Structure (always output this shape)

```
[app_name]/
├── bin/
│   └── [app_name]                       # Main entry point (loop + gum calls)
└── lib/
    ├── gum.sh                           # Internal adapter wrapper functions
    ├── menu.sh                          # Menu logic and dispatch
    └── settings/
        └── config.sh                    # Global styles and settings
```

Rename `app_name` → the actual application name (snake_case for files).

## Internal Adapter Pattern (lib/gum.sh)

To protect against gum CLI flag changes, generate TUI code through a stable internal adapter pattern rather than calling `gum` directly in every file. Define wrapper functions in `lib/gum.sh`:

```bash
#!/usr/bin/env bash

# lib/gum.sh
# Internal adapter — isolates gum CLI surface.
# If gum flags change, update here only.

gum_choose() {
  # Wraps gum choose with project styles
  gum choose "$@" \
    --cursor.foreground="$GUM_CURSOR_COLOR" \
    --item.foreground="$GUM_ITEM_COLOR" \
    --selected.foreground="$GUM_SELECTED_COLOR"
}

gum_input() {
  gum input "$@" \
    --prompt.foreground="$GUM_PROMPT_COLOR" \
    --cursor.foreground="$GUM_CURSOR_COLOR"
}

gum_write() {
  gum write "$@" \
    --base.border="rounded" \
    --base.margin="1" \
    --base.padding="1"
}

gum_spin() {
  local title="$1"
  shift
  gum spin --spinner="dot" --title="$title" -- "$@"
}

gum_confirm() {
  gum confirm "$@" \
    --prompt.foreground="$GUM_PROMPT_COLOR" \
    --selected.background="$GUM_SELECTED_COLOR"
}

gum_filter() {
  gum filter "$@" \
    --indicator="▶" \
    --match.foreground="$GUM_SELECTED_COLOR"
}

gum_table() {
  gum table "$@" \
    --border="rounded" \
    --header.foreground="$GUM_HEADER_COLOR"
}

gum_pager() {
  gum pager "$@"
}
```

All scripts call `gum_choose` and `gum_input` rather than `gum choose` or `gum input` directly.

## Bash/Gum Conventions

- **State** — stored in variables or temporary files. Use `settings/config.sh` for persistence.
- **Main Loop** — Use `while true` in the entry point to handle navigation.
- **Return Pattern** — Functions return data via `echo` and status via `return`.
- **Styling** — All colors and border styles are defined in `settings/config.sh`.

## Patterns Reference

Load `references/tui-patterns.md` for:
- gum choose menu patterns (single select, multi-select)
- gum input with validation
- gum spin for async operations
- gum confirm for destructive action guards
- gum filter for interactive search
- gum table for structured data display
- Combining gum with Return::monad pattern (success/failure output)
- Error handling in gum interactions
- Formatting gum output for piping

## Output Format

For scaffolds:
1. **Full file tree** — every file to be created
2. **Complete content** for each file — logic left as stubs only if
   they require external tool dependencies not yet defined
3. **Boot instruction** — `chmod +x bin/[app_name] && ./bin/[app_name]`
4. **Context7 IDs used** — or WARNING blocks if resolution failed

For advisory:
1. **Direct recommendation** with gum command rationale
2. **Minimal snippet** for the specific pattern
3. **No full scaffold** unless asked
