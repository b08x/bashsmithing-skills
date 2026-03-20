# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Bashsmith is a project template for smithing new Bash shell-based projects with best practices:
convention-aware scripts, JSON return monads, gum-based TUIs, and Shellcheck + BATS quality gates.
This workspace also contains the full Bashsmithing skill suite — AI agent skills for generating,
refactoring, and auditing Bash code.

## Development Commands

```bash
# Run all quality checks (shellcheck + bats)
./bin/qa

# Launch the interactive TUI menu
./bin/run

# Debug mode — enables set -x xtrace
BASHSMITH_DEBUG=1 ./bin/run

# Run a single BATS test file
bats bats/my_test.bats

# Lint specific files manually
shellcheck --exclude=SC1090,SC1091,SC2034,SC2317 lib/monad.sh
```

`bin/qa` lints `bin/`, `lib/`, `settings/`, and `bashsmithing-context/scripts/` then runs all `*.bats` files in `bats/`.

## Execution Model

Every `bin/` entry point must source `settings/main.sh` **first** — this enables strict mode (`set -euo pipefail`, `IFS=$'\n\t'`) and auto-sources `lib/monad.sh`. Then source additional libs as needed:

```bash
source settings/main.sh           # strict mode + monad.sh auto-loaded
source "${BASHSMITH_LIB}/gum.sh"  # TUI adapter (check gum is installed first)
```

The bootstrap guard `[[ "${BASH_SOURCE[0]}" == "${0}" ]]` prevents functions from executing when a file is sourced as a library.

## Return Monad API

`lib/monad.sh` provides structured JSON returns — use these instead of bare `return` or `echo`:

```bash
Return::success "data"           # {"status":"ok","data":"data"}
Return::failure "error msg" 2    # {"status":"error","message":"error msg","code":2}

# Helpers
Return::is_ok "$result"          # returns 0 if status == "ok"
Return::data "$result"           # extracts .data field
Return::message "$result"        # extracts .message field
Return::unwrap "$result" val ok  # sets $val and $ok (0/1) in caller scope
Return::map "$result" "jq '.x'"  # transform .data, pass failures through
Return::chain "$(fn1)" "fn2"     # pipe fn1's .data into fn2, short-circuit on failure
```

## Gum TUI Adapter

`lib/gum.sh` wraps the `gum` CLI — **always use `Gum::*` functions, not raw `gum` calls**, so theme changes only require one edit. Theme is controlled via env vars:

```bash
BASHSMITH_GUM_THEME=catppuccin   # simple (default), base16, catppuccin, dracula
BASHSMITH_GUM_COLOR_PRIMARY=212  # overrides per-color (ANSI 256 or hex)
```

Key wrappers: `Gum::choose_single`, `Gum::input_required`, `Gum::spin`, `Gum::confirm_destructive`, `Gum::filter_single`, `Gum::table_csv`, `Gum::box`, `Gum::success/error/info`.

## Bashsmithing Conventions

- All scripts: `set -euo pipefail` + `IFS=$'\n\t'` via `settings/main.sh`
- All internal state: `local` keyword — no global variable declarations
- Functions: `Namespace::function_name` pattern, two-space indent
- Error handling: `Return::success` / `Return::failure` monads from `lib/monad.sh`
- Never bare `echo` for output — use structured logging or monad returns
- Shellcheck compliance via `.shellcheckrc` (`enable=all` with specific disables)
- No `alias` in scripts (only in interactive shells)
- Quote all expansions: `"$var"` not `$var`
- Use `${var:-default}` for safe defaults
- Subshell wrap environment-altering commands: `( cd /tmp && do_thing )`

## Bashsmithing Skill Suite

| Skill | Purpose |
| :---- | :---- |
| `bashsmithing/` | Hub skill — routes requests, Lite vs Standard mode detection |
| `bashsmithing-context/` | CLI tool docs verification via Context7 |
| `bashsmithing-tui/` | gum-based TUI scaffolding |
| `bashsmithing-refactor/` | Bash anti-pattern enforcement |
| `bashsmithing-report/` | Bash SIFT QA protocol |

**Lite Mode** activates for single-file output ≤ ~50 lines (relaxes namespacing and monad requirements). **Standard Mode** is always used for multi-file scaffolds. Convention target is auto-detected: `.shellcheckrc` → shellcheck enforced; `bats/` → BATS enforced; `lib/monad.sh` → monads enforced.

## Use Context7 MCP for Loading Documentation

Context7 MCP is available to fetch up-to-date documentation with code examples.

**Recommended library IDs**:

- `/charmbracelet/gum` - Terminal UI tools for glamorous shell scripts (choose, input, write, spin, confirm, filter, table, format, pager)
- `/stephencelis/bats-core` - Bash Automated Testing System (BATS) for shell script tests
- `/koalaman/shellcheck` - Shell script linter with SC code documentation
- `/stedolan/jq` - Lightweight JSON processor for Bash
- `/github/gh` - GitHub CLI for repository management
- `/charmbracelet/fish-prompt-template` - Prompt utilities (reference for TUI patterns)
