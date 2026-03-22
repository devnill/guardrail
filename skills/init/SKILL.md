---
description: "Detect project tech stacks and generate tuned permission allowlists and sandbox configuration"
user-invocable: true
argument-hint: "[--no-sandbox]"
---

# Guardrail Init

You are executing the guardrail init skill. Follow these four phases exactly in order. Do not skip any phase.

## Phase 1: Discovery

### 1.1 Load Stack Definitions

1. Use Glob to find all `*.json` files in `${CLAUDE_PLUGIN_ROOT}/stacks/`.
2. Use Glob to find all `*.json` files in `.claude/guardrail/stacks/`. If this directory does not exist, skip silently — project overrides are optional.
3. Build a combined stack list. For files with the **same name** in both locations, the project version **replaces the plugin version entirely** — do not deep-merge them.
4. Read and parse each stack JSON file. Each file conforms to this schema:

```
StackModule: {
  name: string,
  description: string,
  detect: { files: string[], match: "any" | "all" },
  permissions: { allow: string[], deny?: string[] },
  sandbox?: {
    filesystem?: { allowWrite?: string[], denyWrite?: string[] },
    network?: { allowedDomains?: string[] }
  }
}
```

5. If `_base.json` is not found in either location, stop and report an error: "_base.json is a required file and was not found in stacks/. The guardrail plugin may be misconfigured."

### 1.2 Detect Stacks

For each stack **except** `_base`:

1. For each pattern in `detect.files`, use Glob against the project root directory.
2. If `detect.match` is `"any"`: mark the stack as detected if **at least one** pattern matched any file.
3. If `detect.match` is `"all"`: mark the stack as detected only if **every** pattern matched at least one file.
4. Record which file(s) triggered detection — you will display this to the user.

### 1.3 Load Rule Modules

1. Use Glob to find all `*.json` files in `${CLAUDE_PLUGIN_ROOT}/rules/`.
2. Use Glob to find all `*.json` files in `.claude/guardrail/rules/`. If this directory does not exist, skip silently.
3. Same-name project files replace plugin files entirely.
4. Read and parse each rule JSON file. Each file conforms to this schema:

```
RuleModule: {
  name: string,
  description: string,
  auto?: boolean,
  permissions: { allow: string[], deny?: string[] }
}
```

5. Separate rules into two groups:
   - **auto rules**: rules with `"auto": true` — these are always included and cannot be toggled off.
   - **optional rules**: all other rules — these are presented for user selection.

### 1.4 Load Sandbox Template

1. Read `${CLAUDE_PLUGIN_ROOT}/sandbox/default.json`. It conforms to this schema:

```
SandboxTemplate: {
  enabled: boolean,
  filesystem: { allowWrite: string[], denyWrite: string[] },
  network: { allowedDomains: string[] }
}
```

2. Check if the user passed `--no-sandbox` as an argument. If so, sandbox will be disabled.

## Phase 2: Interactive Menu

Present the following menu to the user. Use checkbox-style formatting exactly as shown.

```
Detected stacks:
  [x] <name> (<matched file> found)
  [x] <name> (<matched file> found)

Available stacks:
  [ ] <name>
  [ ] <name>

Rule modules:
  [x] <name> (recommended)
  [ ] <name>
  (auto-included: <comma-separated list of auto rule names>)

Sandbox: [x] enabled
```

Formatting rules:
- Detected stacks appear pre-checked with the triggering filename in parentheses.
- Undetected stacks appear unchecked under "Available stacks".
- Optional rule modules appear as toggleable. Mark rules whose descriptions suggest general utility as `(recommended)` — but the user can change anything.
- Auto-included rules are listed in a parenthetical note and are not toggleable.
- If `--no-sandbox` was passed, show `Sandbox: [ ] disabled`.
- If no stacks were detected at all, display a note: "No stacks auto-detected. You can select stacks manually, or proceed with base permissions only."

After presenting the menu, ask: **"Adjust any selections, or proceed with these?"**

Wait for the user's response. If they request changes, update the selections and re-display the menu. Repeat until the user confirms they want to proceed.

## Phase 3: Merge

Build the final configuration by merging in this exact order:

### 3.1 Merge Permissions

1. **Start with `_base`**: initialize `allow` and `deny` arrays from `_base.json`'s permissions.
2. **Selected stacks**: for each selected stack (in alphabetical order by name), union its `permissions.allow` into the `allow` array and union its `permissions.deny` into the `deny` array.
3. **Selected rules** (including all auto:true rules): for each rule, union its `permissions.allow` into the `allow` array and union its `permissions.deny` into the `deny` array.
4. **Deduplicate** both arrays — remove exact duplicates.
5. **Sort** both arrays alphabetically.

### 3.2 Merge Sandbox

If sandbox is enabled:

1. Start from `sandbox/default.json` as the base.
2. For each selected stack that has a `sandbox` key:
   - Union `sandbox.filesystem.allowWrite` arrays.
   - Union `sandbox.filesystem.denyWrite` arrays.
   - Union `sandbox.network.allowedDomains` arrays.
3. Deduplicate all arrays.
4. Sort all arrays alphabetically.

If sandbox is disabled (`--no-sandbox`), omit the `sandbox` key entirely from the output.

### 3.3 Assemble Output Object

Build the final JSON object with exactly these top-level keys (and no others):

- `permissions`: `{ "allow": [...], "deny": [...] }` — omit `deny` if the array is empty.
- `sandbox`: `{ "enabled": true, "filesystem": { "allowWrite": [...], "denyWrite": [...] }, "network": { "allowedDomains": [...] } }` — only if sandbox is enabled.

## Phase 4: Write

### 4.1 Determine Target Path

1. Run: `git rev-parse --git-dir 2>/dev/null`
   - If this **succeeds**: the project is in a git repo. Derive the git root (parent of the `.git` dir) and set target to `<git-root>/.claude/settings.local.json`.
   - If this **fails**: the project is not in a git repo. Ask the user for a target path, defaulting to `<cwd>/.claude/settings.local.json`.

### 4.2 Handle Existing File

If the target file already exists:

1. Read and parse it.
2. If it contains invalid JSON, warn the user: "Existing settings file contains invalid JSON. Cannot merge safely." Then ask whether to back up and replace, or abort.
3. Extract the current `permissions` and `sandbox` keys from the existing file.
4. Show a diff to the user:
   - Permissions that would be **added** (in new but not in existing).
   - Permissions that would be **removed** (in existing but not in new).
   - Sandbox changes.
5. Ask: **"Merge (add new rules to existing) or Replace (overwrite permissions + sandbox)?"**
   - **Merge**: union the new `allow`/`deny` arrays with the existing ones. Deduplicate and sort. Union sandbox arrays similarly.
   - **Replace**: overwrite `permissions` and `sandbox` keys with the new values. Create a backup of the existing file at `settings.local.json.bak` before replacing.
6. **Preserve all other keys** in the existing file (e.g., `env`, `hooks`, `plugins`, `mcpServers`, or any other keys). Only touch `permissions` and `sandbox`.

### 4.3 Create Directory If Needed

If the target file does not exist and the `.claude/` directory does not exist, create it.

### 4.4 Confirm and Write

1. Show the **complete final JSON** that will be written to the target path.
2. Ask for explicit confirmation: **"Write this configuration to `<target-path>`? (yes/no)"**
3. Only if the user confirms with "yes": write the file using the Write tool.
4. After writing, display: "Configuration written to `<target-path>`."

## Edge Cases

- **Empty project directory** (no stacks detected): still include `_base` permissions plus any selected rules. This is valid — the user may want base permissions only.
- **Existing settings with non-JSON content**: warn the user and do not overwrite unless they explicitly choose to back up and replace.
- **`.claude/guardrail/` directory does not exist**: skip project overrides silently. Do not warn or error.
- **`_base.json` missing**: this is a fatal error. Stop and report it. Do not proceed with the remaining phases.
- **No rules selected and no stacks detected**: still write `_base` permissions. This is the minimal valid configuration.
