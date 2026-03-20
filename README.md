# 🛠️ Bashsmithing Skills

[![Shellcheck](https://img.shields.io/badge/shellcheck-passed-brightgreen.svg)](https://www.shellcheck.net/)
[![BATS](https://img.shields.io/badge/tests-BATS-blue.svg)](https://github.com/bats-core/bats-core)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Bash](https://img.shields.io/badge/bash-4.0%2B-orange.svg)](https://www.gnu.org/software/bash)

**Bashsmithing Skills** is a specialized AI-powered skill suite and project template for crafting robust, production-ready Bash scripts. It bridges the gap between "quick-and-dirty" scripts and reliable, maintainable software by enforcing strict-mode settings, JSON-based return monads, and a modular TUI architecture.

> [!NOTE]
> This project is a specialized derivative based on the [Bashsmith](https://github.com/bkuhlmann/bashsmith) template. While it inherits the core architecture and philosophy of the original, it is maintained as a separate project focused on the Bashsmithing AI skill suite.

---

## 🌟 Key Features

*   **🛡️ Strict-Mode Infrastructure**: Automatic enforcement of `set -euo pipefail` and custom `IFS` across all entry points.
*   **📦 JSON Return Monads**: A structured `lib/monad.sh` that replaces fragile exit codes with predictable, parseable JSON status objects.
*   **🎨 Elegant TUI**: A built-in `lib/gum.sh` adapter for [Gum](https://github.com/charmbracelet/gum), providing beautiful menus, inputs, and spinners with theme support.
*   **✅ Quality Gates**: Integrated `bin/qa` script for comprehensive Shellcheck linting and BATS test execution.
*   **🤖 AI Skill Suite**: Five specialized agent skills for generating, refactoring, and auditing Bash code.

---

## 🚀 Quick Start

### Prerequisites

Ensure you have the following tools installed:

```bash
# MacOS (Homebrew)
brew install charmbracelet/tap/gum jq shellcheck bats-core

# Linux (Debian/Ubuntu example)
sudo apt install jq shellcheck
# Install gum and bats-core via their respective install scripts
```

### Installation

```bash
git clone https://github.com/b08x/bashsmithing-skills.git
cd bashsmithing-skills
./bin/run
```

---

## 🏗️ Architecture

Every Bashsmithing script follows a standardized three-step bootstrap process:

1.  **Sourcing Settings**: `source settings/main.sh` enables strict mode and auto-loads the monad library.
2.  **Loading Libraries**: Source specialized modules like `lib/gum.sh` or your own custom logic.
3.  **Bootstrap Guard**: Use `[[ "${BASH_SOURCE[0]}" == "${0}" ]]` to ensure functions only execute when the script is run directly.

### The Return Monad Pattern

Stop guessing what your functions returned. Use structured JSON:

```bash
# Emit a success monad
Return::success "Operation completed." 

# Emit a failure monad
Return::failure "Something went wrong." 1

# Unwrap and handle
result=$(my_function)
if Return::is_ok "$result"; then
  echo "Data: $(Return::data "$result")"
else
  echo "Error: $(Return::message "$result")"
fi
```

---

## 🛠️ Development & QA

Maintain high standards with built-in quality tools:

| Command | Action |
| :--- | :--- |
| `./bin/run` | Launch the interactive TUI menu |
| `./bin/qa` | Run all quality checks (Shellcheck + BATS) |
| `bats bats/` | Run all tests in the `bats/` directory |
| `BASHSMITH_DEBUG=1 ./bin/run` | Enable verbose `xtrace` debugging |

---

## 🧠 AI Agent Skill Suite

Bashsmithing isn't just a template; it's a suite of specialized skills for your AI agents:

*   **`bashsmithing/`**: The hub skill for project scaffolding.
*   **`bashsmithing-tui/`**: Specialized in building TUI applications.
*   **`bashsmithing-refactor/`**: Identifies and fixes Bash anti-patterns.
*   **`bashsmithing-report/`**: Generates SIFT QA protocol reports.
*   **`bashsmithing-context/`**: Verified CLI documentation lookup.

---

## 📄 License & Attribution

This project is licensed under the **MIT License**.

### Credits

This project is forked from and inspired by the original [Bashsmith](https://github.com/bkuhlmann/bashsmith) project by [Brooke Kuhlmann](https://www.alchemists.io). 

The original template components (located in `LICENSE.adoc`) are subject to the [Hippocratic License 2.1.0](LICENSE.adoc). Users of this derivative project are encouraged to review the ethical standards set forth in that license.

© 2026 [Robert Pannick](https://github.com/b08x) & Contributors.
