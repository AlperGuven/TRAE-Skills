---
name: "agent-browser"
description: "Browser automation CLI for UI testing, E2E tests, and page interaction. Invoke when user wants to test UI, click buttons, fill forms, or take screenshots."
---

# Agent Browser

Native Rust CLI for browser automation. Lightweight alternative to Playwright.

## Setup

```bash
npm install -g agent-browser
agent-browser install  # First time only
```

## Core Commands

| Command | Description |
|---------|-------------|
| `agent-browser open <url>` | Navigate to URL |
| `agent-browser snapshot` | Get accessibility tree with refs |
| `agent-browser screenshot [path]` | Take screenshot |
| `agent-browser click <sel>` | Click element |
| `agent-browser fill <sel> <text>` | Fill input |
| `agent-browser close` | Close browser |
| `agent-browser find <criteria>` | Find element by role/text/label |
| `agent-browser keyboard type <key>` | Send keyboard input |
| `agent-browser scroll <direction> <px>` | Scroll page |
| `agent-browser mouse <action>` | Mouse operations (move, down, up, wheel) |

## Trae Sandbox Notes

In sandbox environment, `~/.agent-browser` socket directory has write permission issues. Use this prefix for all commands:

```bash
# Prefix every command with HOME=/tmp:
HOME=/tmp agent-browser <command>

# Examples:
HOME=/tmp agent-browser open http://localhost:5173
HOME=/tmp agent-browser snapshot
HOME=/tmp agent-browser click @e5
```

## Combobox/Dropdown Click Issue

`click @ref` may not work on Bootstrap custom select dropdown options. Solutions:

```bash
# Solution 1: find with text match
HOME=/tmp agent-browser find text "Kg" click

# Solution 2: nth for specific selection (when multiple matches exist)
HOME=/tmp agent-browser find nth 1 text "Kg" click

# Solution 3: Keyboard navigation
HOME=/tmp agent-browser click @e44              # Open combobox
HOME=/tmp agent-browser keyboard type "key ArrowDown"  # First option
HOME=/tmp agent-browser keyboard type "Enter"           # Select
```

## Verification Strategy

```bash
# After each step, check snapshot
HOME=/tmp agent-browser snapshot | grep "validation message"

# Take screenshot if unsure
HOME=/tmp agent-browser screenshot ./test-step-X.png

# Refs may change - check each time
HOME=/tmp agent-browser snapshot | grep -A 5 "element name"

# Check dialog content
HOME=/tmp agent-browser snapshot | grep -A 30 "dialog"
```

## Selector Types

- **Ref**: `@e1`, `@e2` (from snapshot)
- **CSS**: `#id`, `.class`, `button`
- **Text**: `button:has-text("Submit")`
- **Role**: `find role button click --name "Submit"`
- **Text match**: `find text "Kg" click`
- **Nth selection**: `find nth 1 text "Kg" click`

## Usage Flow

1. **Open and close browser**
   ```bash
   # Close existing browser if any
   HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1

   # Open page
   HOME=/tmp agent-browser open http://localhost:5173/retailer/application/create
   ```

2. **Inspect elements**
   ```bash
   HOME=/tmp agent-browser snapshot  # Shows refs like @e1, @e2
   ```

3. **Interact**
   ```bash
   HOME=/tmp agent-browser click @e5                      # Click by ref
   HOME=/tmp agent-browser find text "Kg" click           # Click by text
   HOME=/tmp agent-browser fill @e10 "value"             # Fill input
   HOME=/tmp agent-browser scroll down 300                # Scroll
   ```

4. **Verify**
   ```bash
   HOME=/tmp agent-browser screenshot ./test-step-X.png   # Screenshot
   HOME=/tmp agent-browser snapshot | grep "message"     # Content check
   ```

5. **Cleanup**
   ```bash
   HOME=/tmp agent-browser close
   ```

## Examples

### Retailer Panel - Login

```bash
HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1
HOME=/tmp agent-browser open http://localhost:5173/login
HOME=/tmp agent-browser snapshot

# Form fields (refs may change, check with snapshot)
HOME=/tmp agent-browser fill input[type='email'] "basak@tarim.com"
HOME=/tmp agent-browser fill input[type='password'] "basaktarim123"
HOME=/tmp agent-browser click "button[type='submit']"

# Check redirect after login
sleep 2
HOME=/tmp agent-browser screenshot ./retailer-login.png
HOME=/tmp agent-browser snapshot | head -20
```

### Retailer Panel - Add Product

```bash
HOME=/tmp agent-browser open http://localhost:5173/retailer/application/create
HOME=/tmp agent-browser snapshot

# Find and click "Add product" button
HOME=/tmp agent-browser find text "Add product" click
sleep 1

# Check if modal opened
HOME=/tmp agent-browser screenshot ./modal-open.png
HOME=/tmp agent-browser snapshot | grep -A 30 "dialog"

# Search product
HOME=/tmp agent-browser click @e49  # Product search box
HOME=/tmp agent-browser type @e49 "wheat"
sleep 1
HOME=/tmp agent-browser find text "Wheat" click

# Select brand (while dropdown is open)
HOME=/tmp agent-browser find text "Brand" click
sleep 1
HOME=/tmp agent-browser find nth 1 text "Abalim" click

# Select unit
HOME=/tmp agent-browser find text "Unit" click
sleep 1
HOME=/tmp agent-browser find text "Kg" click

# Fill quantity and price
HOME=/tmp agent-browser fill @e45 "10"
HOME=/tmp agent-browser fill @e46 "100"

# Verify form
HOME=/tmp agent-browser snapshot | grep "Add"
HOME=/tmp agent-browser screenshot ./product-form-filled.png
```

### Admin Panel - Login

```bash
HOME=/tmp agent-browser close --all 2>/dev/null; sleep 1
HOME=/tmp agent-browser open http://localhost:5173/admin/login
HOME=/tmp agent-browser fill input[type='email'] "alper.guven@tarfin.com"
HOME=/tmp agent-browser fill input[type='password'] "93209320Tt"
HOME=/tmp agent-browser click "button[type='submit']"
sleep 2
HOME=/tmp agent-browser screenshot ./admin-login.png
```

## Requirements

- Chrome for Testing (`agent-browser install`)
- No Node.js/Playwright dependency needed for the daemon

## Troubleshooting

| Problem | Solution |
|---------|----------|
| Socket permission error | Use `HOME=/tmp agent-browser` |
| Element not found | Check ref with snapshot |
| Click not working | Try `find text` or `keyboard` |
| Dropdown not opening | Click on combobox first, then use `find text` to select option |
| Form not submitting | Check validation messages (`snapshot \| grep "required"`) |
