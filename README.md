# Bashsmith

**Description**: A project template and AI skill suite for production-grade Bash scripting.

Bashsmith is a project template and AI skill suite for writing production-grade Bash scripts. It gives you a working scaffold with strict-mode settings, JSON return monads, a gum TUI adapter, and Shellcheck + BATS quality gates — plus a set of AI agent skills for generating, refactoring, and auditing Bash code.

## Requirements

- [Bash 4.0+](https://www.gnu.org/software/bash)
- [gum](https://github.com/charmbracelet/gum) — for TUI interactions (`brew install charmbracelet/tap/gum`)
- [jq](https://stedolan.github.io/jq) — for monad unwrapping (`brew install jq`)
- [shellcheck](https://www.shellcheck.net) — for linting (`brew install shellcheck`)
- [bats](https://github.com/bats-core/bats-core) — for tests (`brew install bats-core`)

## Setup

```bash
git clone https://github.com/bkuhlmann/bashsmith.git
cd bashsmith
```

To use as a template for a new project, clone and replace `Bashsmith::` with your own namespace throughout `bin/` and `lib/`.

## Running & Testing

```bash
# Launch the interactive TUI menu
./bin/run

# Run all quality checks (shellcheck + bats)
./bin/qa

# Run a single BATS test file
bats bats/my_test.bats

# Enable xtrace debugging
BASHSMITH_DEBUG=1 ./bin/run
```

`bin/qa` lints `bin/`, `lib/`, `settings/`, and `bashsmithing-context/scripts/`, then runs all `*.bats` files in `bats/`.

## Framework Architecture

Every entry point follows the same three-step bootstrap:

```bash
#!/usr/bin/env bash
source settings/main.sh           # <1>
source \