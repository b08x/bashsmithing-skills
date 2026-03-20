# TUI Patterns (Gum)

Standardized patterns for building terminal UIs in bashsmithing projects using
**gum** (Charm's CLI tools). All gum calls should go through the `Gum::*` adapter
in `lib/gum.sh` — never call `gum` directly. This isolates gum's CLI surface so
API or flag changes only need updating in one place.

> **Verify gum flags** before generating code: activate `bashsmithing-context`
> and check `/charmbracelet/gum` for the latest syntax.

---

## gum Adapter (`lib/gum.sh`)

The adapter wraps the gum CLI with theme defaults and Bash-friendly error handling.

| Adapter function | Wraps gum | Purpose |
|---|---|---|
| `Gum::choose_single` | `gum choose` | Single selection menu |
| `Gum::choose_multi` | `gum choose --no-limit` | Multi-selection |
| `Gum::choose_from_file` | `gum choose < file` | Single from file |
| `Gum::menu_main` | `gum choose` | Titled menu with descriptions |
| `Gum::input` | `gum input` | Single-line text input |
| `Gum::input_required` | `gum input` + loop | Non-empty validation |
| `Gum::write` | `gum write` | Multi-line input (Ctrl+D) |
| `Gum::spin` | `gum spin` | Spinner during long task |
| `Gum::confirm` | `gum confirm` | Yes/no prompt |
| `Gum::confirm_destructive` | `gum confirm` | Styled destructive guard |
| `Gum::filter_single` | `gum filter` | Fuzzy single select |
| `Gum::filter_multi` | `gum filter` | Fuzzy multi select |
| `Gum::table_csv` | `gum table` | Render CSV table |
| `Gum::box` | `gum style` | Styled text box |
| `Gum::header` | `gum style` | Section header |
| `Gum::pager` | `gum pager` | Scrollable content |

---

## 1. Main Menu Loop

The application entry point. Uses `gum choose` for the main menu with
fallback to a read-based menu if gum is not installed.

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

source settings/main.sh
source "${BASHSMITH_LIB}/gum.sh"
source "${BASHSMITH_LIB}/monad.sh"

# Menu items: "label:description"
MENU_ITEMS=(
  "scan:Scan for open ports"
  "deploy:Deploy to environment"
  "logs:Tail application logs"
  "config:Edit configuration"
  "quit:Exit"
)

# Trap Ctrl+C cleanly.
Gum::catch_ctrlc

# Build the menu from items.
build_menu() {
  local title="${1:?Title required}"; shift
  printf '%s\n' "$@" | gum choose --header "$title"
}

main() {
  while true; do
    clear
    Gum::header "NetScan CLI"

    local choice
    choice=$(printf '%s\n' "${MENU_ITEMS[@]}" | gum choose --cursor "> " \
      --header "Select an action:" \
      --selected.foreground 82)

    local action="${choice%%:*}"

    case "$action" in
      scan)    run_scan ;;
      deploy)  run_deploy ;;
      logs)    run_logs ;;
      config)  edit_config ;;
      quit)    Gum::success "Goodbye!"; break ;;
      *)       Gum::error "Unknown action: $action" ;;
    esac

    # Pause so user can read output before the next menu.
    gum input --placeholder "Press Enter to continue..." > /dev/null
  done
}

main "$@"
```

---

## 2. gum choose — Interactive Menus

### Single selection (most common)

```bash
local choice
choice=$(gum choose "Deploy" "Rollback" "Cancel")
if [[ -z "$choice" ]]; then
  Gum::error "No selection made."
  return 1
fi
```

### Single selection from an array

```bash
local -a OPTIONS=("Production" "Staging" "Development")
local env
env=$(printf '%s\n' "${OPTIONS[@]}" | gum choose --header "Select environment:")
```

### Multi-selection with limit

```bash
# Select up to 5 servers
local servers
servers=$(gum choose --limit 5 \
  "server-01" "server-02" "server-03" "server-04" "server-05" \
  "server-06" "server-07" "server-08")
```

### Multi-selection with all allowed

```bash
local plugins
plugins=$(cat plugins.txt | gum choose --no-limit --header "Select plugins to install:")
echo "$plugins" | while IFS= read -r plugin; do
  echo "Installing: $plugin"
done
```

### Pre-selected options

```bash
local langs
langs=$(gum choose \
  --selected "Ruby" \
  --selected "Bash" \
  "Ruby" "Bash" "Python" "Go" "Rust")
```

### Custom cursor and highlight colors

```bash
local choice
choice=$(gum choose \
  --cursor "> " \
  --cursor.foreground 212 \
  --selected.foreground 82 \
  "Option A" "Option B" "Option C")
```

---

## 3. gum input — Text Entry

### Basic text input

```bash
local name
name=$(gum input --placeholder "Enter your name")
echo "Hello, $name"
```

### Input with pre-filled default

```bash
local host
host=$(gum input \
  --placeholder "Hostname" \
  --value "localhost" \
  --prompt "Host: ")
```

### Password input (masked)

```bash
local password
password=$(gum input \
  --password \
  --placeholder "API key" \
  --prompt "Key: ")
```

### Required input with validation loop

```bash
local email
while true; do
  email=$(gum input --placeholder "user@example.com" --prompt "Email: ")
  if [[ "$email" =~ ^[A-Za-z0-9._%+-]+@[A-Za-z0-9.-]+\.[A-Za-z]{2,}$ ]]; then
    break
  fi
  Gum::error "Invalid email. Try again."
done
echo "Got: $email"
```

### Multiple inputs in sequence

```bash
local name email role
name=$(gum input --placeholder "Full name")
email=$(gum input --placeholder "Email address")
role=$(printf '%s\n' "Admin" "User" "Guest" | gum choose --header "Role:")

cat <<EOF
Name:  $name
Email: $email
Role:  $role
EOF
```

---

## 4. gum write — Multi-line Input

### Basic multi-line (Ctrl+D to finish)

```bash
local description
description=$(gum write --placeholder "Enter a description...")

# Save to file
echo "$description" > description.txt
```

### Multi-line with character limit

```bash
local changelog
changelog=$(gum write \
  --placeholder "Changelog entry..." \
  --char-limit 280)
```

### Pre-filled multi-line input

```bash
local commit_msg
commit_msg=$(gum write \
  --value "Fix: correct edge case in user auth

BREAKING CHANGE: drops support for legacy token format
" \
  --placeholder "Edit commit message...")
```

---

## 5. gum spin — Long-running Tasks

### Basic spinner

```bash
Gum::spin "Scanning ports..." nmap -sS localhost
```

### With spinner style

```bash
local result
result=$(Gum::spin_custom \
  "Cloning repository..." \
  globe \
  git clone "$repo_url" /tmp/repo)
```

### Spin with visible command output

```bash
gum spin \
  --title "Installing packages..." \
  --show-output \
  --spinner dot \
  npm install
```

### Multiple parallel spinners (background jobs + gum spin for each)

```bash
# Spin while a command runs; combine with Return monads
result=$(Gum::spin "Fetching config..." \
  bash -c 'curl -s https://api.example.com/config')
if Return::is_ok "$result"; then
  config=$(Return::data "$result")
fi
```

---

## 6. gum confirm — Action Guards

### Basic confirmation

```bash
if gum confirm "Deploy to production?"; then
  Gum::spin "Deploying..." deploy.sh --env production
  Gum::success "Deployed successfully."
else
  Gum::info "Deployment cancelled."
fi
```

### Guard for destructive action

```bash
Gum::confirm_destructive() {
  local action="${1:?Action required}"
  gum confirm \
    --affirmative "Delete" \
    --negative "Cancel" \
    "$(gum style --foreground 196 "⚠  Delete: $action")"
}

if Gum::confirm_destructive "ALL DATA in /var/data"; then
  rm -rf /var/data
fi
```

### Chained: confirm then proceed

```bash
gum confirm "Commit changes?" && \
  git commit -m "$SUMMARY" -m "$DESCRIPTION"
```

---

## 7. gum filter — Fuzzy Search

### Filter from array

```bash
local file
file=$(printf '%s\n' "${all_files[@]}" | \
  gum filter --limit 1 --placeholder "Search files...")
echo "Editing: $file"
```

### Filter git branches

```bash
local branch
branch=$(git branch | cut -c 3- | \
  gum filter --limit 1 --placeholder "Switch to branch...")
git checkout "$branch"
```

### Multi-select with unlimited

```bash
local selected
selected=$(cat <<'EOF' | gum filter --no-limit --header "Select packages:"
bashsmith
rubysmithing
bashsmithing-context
bashsmithing-tui
bashsmithing-refactor
bashsmithing-report
EOF
)
echo "Selected: $selected"
```

### Filter with custom styling

```bash
local choice
choice=$(cat packages.txt | gum filter \
  --indicator "→ " \
  --match.foreground 212 \
  --placeholder "Search packages...")
```

---

## 8. gum table — Structured Data

### Basic table from CSV

```bash
# data.csv: Name,IP,Status
# Strawberry,10.0.0.1,Active
# Banana,10.0.0.2,Active
# Cherry,10.0.0.3,Maintenance

gum table < data.csv
```

### Table with custom column widths and styling

```bash
gum table \
  --columns "Name,IP,Status" \
  --widths 20,15,12 \
  --border rounded \
  --border.foreground 57 \
  --foreground 212 \
  < data.csv
```

### Print all rows without selection (no cursor)

```bash
gum table --print < data.csv
```

### Table from JSON (via jq)

```bash
# Extract fields and format as CSV for gum table
cat data.json | jq -r '.[] | [.name, .ip, .status] | @csv' | \
  gum table --columns "Name,IP,Status"
```

---

## 9. gum style — Text Styling

### Colored text

```bash
gum style --foreground 196 "Error: something went wrong"
gum style --foreground 82 "Success: deployed"
gum style --foreground 240 --italic "This is muted and italic"
```

### Bordered box

```bash
BOX=$(gum style \
  --border rounded \
  --border-foreground 212 \
  --padding "1 2" \
  --width 40 \
  "Ready to deploy")
echo "$BOX"
```

### Header separator

```bash
gum style --bold --foreground 212 "══ Deploy Menu ══"
```

### Multi-line styled block

```bash
gum style \
  --foreground 212 --border double --border-foreground 212 \
  --align center --width 50 --margin "1 2" --padding "2 4" \
  "Bubble Gum (1¢)" "So sweet and so fresh!"
```

---

## 10. gum pager — Scrollable Content

### Basic paging

```bash
gum pager < README.md
```

### With line numbers

```bash
gum pager --show-line-numbers < CHANGELOG.md
```

### Soft-wrap long lines

```bash
gum pager --soft-wrap < log_file.txt
```

### With search highlight

```bash
gum pager \
  --match-style "foreground:196,bold" \
  < output.log
```

---

## 11. Combining Gum with Return Monads

gum outputs data via stdout. Wrap gum calls with Return monads for
structured error handling.

### gum input → monad

```bash
Prompt::ask_name() {
  local prompt="${1:-Name:}"
  local name
  name=$(gum input --placeholder "Your name" --prompt "$prompt") || {
    Return::failure "gum input interrupted"
    return 1
  }
  [[ -z "$name" ]] && Return::failure "name cannot be empty"
  Return::success "$name"
}
```

### gum spin → monad

```bash
Task::run_scan() {
  local target="${1:?Target required}"
  local result

  # Capture spinner output
  result=$(gum spin \
    --title "Scanning ${target}..." \
    --spinner dot \
    nmap -sS -T4 "$target" 2>&1) || {
    Return::failure "scan failed: $result"
    return 1
  }

  Return::success "$result"
}
```

### gum confirm → monad

```bash
Guard::destructive() {
  local action="${1:?Action required}"
  gum confirm \
    --affirmative "Proceed" \
    --negative "Abort" \
    "$(gum style --foreground 196 "⚠  $action — are you sure?")" \
    && Return::success "confirmed" \
    || Return::failure "aborted by user"
}
```

---

## 12. Exit Handling — Ctrl+C and Empty Input

### Trap Ctrl+C during gum interaction

```bash
trap 'echo ""; gum style --foreground 196 "Interrupted."; exit 130' INT

choice=$(gum choose "A" "B" "C")
# If user presses Ctrl+C here, the trap fires cleanly.
```

### Check for empty selection (user pressed Esc or Enter with no choice)

```bash
choice=$(gum choose "A" "B" "C" 2>/dev/null)
if [[ -z "$choice" ]]; then
  Gum::info "No selection — returning to menu."
  return 0
fi
```

### Read fallback when gum is unavailable

```bash
get_choice() {
  if command -v gum &>/dev/null; then
    gum choose "$@"
  else
    # Plain read fallback
    local -a options=("$@")
    local i=1
    for opt in "$@"; do
      echo "  $i. $opt"
      i=$((i + 1))
    done
    read -r num
    echo "${options[$((num - 1))]:-}"
  fi
}
```

---

## 13. Theme Configuration

Set theme globally via env vars in `settings/main.sh`:

```bash
# Theme: simple, catppuccin, Dracula, fourk (default: simple)
export BASHSMITH_GUM_THEME="${BASHSMITH_GUM_THEME:-simple}"

# Color overrides (ANSI 256 color codes)
export BASHSMITH_GUM_COLOR_PRIMARY="${BASHSMITH_GUM_COLOR_PRIMARY:-212}"
export BASHSMITH_GUM_COLOR_SECONDARY="${BASHSMITH_GUM_COLOR_SECONDARY:-57}"
export BASHSMITH_GUM_COLOR_ERROR="${BASHSMITH_GUM_COLOR_ERROR:-196}"
export BASHSMITH_GUM_COLOR_SUCCESS="${BASHSMITH_GUM_COLOR_SUCCESS:-82}"
```

Or override per-call with gum's own flags:

```bash
gum choose \
  --cursor.foreground 99 \
  --selected.foreground 82 \
  --header.foreground 240 \
  "A" "B" "C"
```

---

## 14. Complete Scaffold Skeleton

For new TUI projects, produce this directory layout:

```
myproject/
├── bin/
│   └── run              # Entry point — sources lib + Gum::main
├── lib/
│   ├── gum.sh           # Gum adapter (always source this)
│   ├── monad.sh         # Return::success / Return::failure
│   └── menu.sh          # Menu definitions
├── settings/
│   └── main.sh          # BASHSMITH_GUM_THEME + paths
└── bats/
    └── run_test.bats   # BATS tests for each Gum::* function
```

The `bin/run` entry point template:

```bash
#!/usr/bin/env bash
set -euo pipefail
IFS=$'\n\t'

BASHSMITH_ROOT="${BASHSMITH_ROOT:-$(cd -- "$(dirname -- "${BASH_SOURCE[0]}")/.." && pwd)}"
export BASHSMITH_ROOT
export BASHSMITH_LIB="${BASHSMITH_LIB:-${BASHSMITH_ROOT}/lib}"
export BASHSMITH_SETTINGS="${BASHSMITH_SETTINGS:-${BASHSMITH_ROOT}/settings}"

source "${BASHSMITH_SETTINGS}/main.sh"
source "${BASHSMITH_LIB}/monad.sh"
source "${BASHSMITH_LIB}/gum.sh"

main() {
  local choice
  Gum::catch_ctrlc

  MENU_ITEMS=(
    "item1:Do thing one"
    "item2:Do thing two"
    "quit:Exit"
  )

  choice=$(printf '%s\n' "${MENU_ITEMS[@]}" | gum choose --header "My App")
  local action="${choice%%:*}"

  case "$action" in
    item1) Gum::spin "Working..." my_command ;;
    item2) my_other_command ;;
    quit)  return 0 ;;
    *)     Gum::error "Unknown: $action" ;;
  esac
}

main "$@"
```
