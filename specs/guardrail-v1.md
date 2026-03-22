# Guardrail Plugin — v1 Specification

## Overview

Guardrail is a Claude Code plugin that automates the configuration of permission allowlists and sandbox settings for projects. It detects project tech stacks, generates tuned permission rules, and audits existing permission configurations to consolidate one-off rules into generalized patterns.

### Problem Statement

Claude Code's permission system prompts users for approval on every new command variant. Over time, project `settings.local.json` files accumulate hundreds of highly specific, non-generalizable rules (e.g., exact file paths, loop fragments, one-off commands). This creates two problems:

1. **Friction during development** — Constant permission prompts interrupt autonomous agent workflows, especially in orchestrated multi-agent systems like ideate.
2. **Permission debt** — Accumulated rules are noisy, hard to audit, and provide no meaningful security benefit since they're too specific to match future commands.

Guardrail solves this by generating **generalized, stack-aware permission rules** and providing tools to **audit and consolidate** existing permission configurations.

### Goals

- Reduce permission prompts by 80%+ for typical development workflows
- Maintain meaningful security boundaries (deny destructive operations, restrict out-of-scope paths)
- Support extensible stack and rule definitions without code changes
- Provide interactive confirmation before writing any configuration

### Non-Goals (v1)

- Hook-based policy engine (future `/guardrail policy`)
- Runtime permission monitoring or analytics
- Auto-detection of MCP servers not already registered
- Managing global `~/.claude/settings.json` (project-scoped only in v1)

---

## Architecture

### Plugin Structure

```
guardrail/
├── .claude-plugin/
│   ├── plugin.json                 # Plugin manifest
│   └── marketplace.json            # Marketplace registry entry
├── skills/
│   ├── init/
│   │   └── SKILL.md               # /guardrail init skill
│   └── audit/
│       └── SKILL.md               # /guardrail audit skill
├── stacks/                         # Stack detection modules (extensible)
│   ├── _base.json                  # Universal safe defaults
│   ├── python.json
│   ├── node.json
│   ├── rust.json
│   ├── go.json
│   └── shell.json
├── rules/                          # Non-stack permission modules (extensible)
│   ├── git.json
│   ├── web.json
│   ├── docker.json
│   ├── mcp-ideate.json
│   └── mcp-cyberbrain.json
├── sandbox/                        # Sandbox configuration templates
│   └── default.json
├── scripts/
│   └── detect-stack.sh             # Stack detection logic
├── CLAUDE.md
└── README.md
```

### Design Principles

1. **Data-driven, not code-driven** — Stack definitions and rule modules are JSON files. Adding support for a new stack or rule set means adding a new JSON file, not modifying plugin code.
2. **Composable** — Multiple stacks and rule modules combine additively. A Python + Docker project gets both permission sets merged.
3. **Extensible** — Third parties (or the user) can drop custom JSON files into `stacks/` or `rules/` to extend guardrail without forking.
4. **Conservative defaults** — The base ruleset allows only read-only and non-destructive operations. Stack modules add stack-specific commands. Nothing allows `rm -rf /`, force push, or operations outside the project directory.
5. **Confirm before write** — All configuration changes are previewed and require explicit user confirmation.

---

## Module Definitions

### Stack Module Schema (`stacks/*.json`)

Each stack module defines detection heuristics and the permissions it contributes.

```json
{
  "name": "python",
  "description": "Python development stack",
  "detect": {
    "files": ["requirements.txt", "pyproject.toml", "setup.py", "Pipfile", "*.py"],
    "match": "any"
  },
  "permissions": {
    "allow": [
      "Bash(python3:*)",
      "Bash(python:*)",
      "Bash(pip3:*)",
      "Bash(pip:*)",
      "Bash(pytest:*)",
      "Bash(mypy:*)",
      "Bash(ruff:*)",
      "Bash(black:*)",
      "Bash(isort:*)",
      "Bash(poetry:*)",
      "Bash(pdm:*)",
      "Bash(uv:*)",
      "Bash(tox:*)",
      "Bash(nox:*)"
    ]
  },
  "sandbox": {
    "network": {
      "allowedDomains": ["pypi.org", "files.pythonhosted.org"]
    }
  }
}
```

**Schema fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Human-readable stack identifier |
| `description` | string | One-line description |
| `detect.files` | string[] | Glob patterns — if any/all match, stack is detected |
| `detect.match` | `"any"` \| `"all"` | Whether any or all file patterns must match |
| `permissions.allow` | string[] | Permission allow rules to add |
| `permissions.deny` | string[] | (Optional) Permission deny rules to add |
| `sandbox.filesystem` | object | (Optional) Sandbox filesystem overrides |
| `sandbox.network` | object | (Optional) Sandbox network overrides |

### Rule Module Schema (`rules/*.json`)

Rule modules are stack-independent permission sets for cross-cutting concerns.

```json
{
  "name": "git",
  "description": "Git version control operations",
  "permissions": {
    "allow": [
      "Bash(git:*)"
    ],
    "deny": [
      "Bash(git push --force:*)",
      "Bash(git reset --hard:*)"
    ]
  }
}
```

```json
{
  "name": "web",
  "description": "Web search and documentation fetching",
  "permissions": {
    "allow": [
      "WebSearch",
      "WebFetch(domain:docs.anthropic.com)",
      "WebFetch(domain:github.com)",
      "WebFetch(domain:stackoverflow.com)"
    ]
  }
}
```

**Schema fields:**

| Field | Type | Description |
|-------|------|-------------|
| `name` | string | Module identifier |
| `description` | string | One-line description |
| `auto` | boolean | (Optional) If true, always included without prompting. Default: false |
| `permissions.allow` | string[] | Permission allow rules |
| `permissions.deny` | string[] | (Optional) Permission deny rules |

### Base Module (`stacks/_base.json`)

Always included. Provides universally safe defaults.

```json
{
  "name": "_base",
  "description": "Universal safe defaults for all projects",
  "detect": {
    "files": ["*"],
    "match": "any"
  },
  "permissions": {
    "allow": [
      "Read",
      "Bash(ls:*)",
      "Bash(cat:*)",
      "Bash(head:*)",
      "Bash(tail:*)",
      "Bash(wc:*)",
      "Bash(find:*)",
      "Bash(grep:*)",
      "Bash(echo:*)",
      "Bash(which:*)",
      "Bash(mkdir:*)",
      "Bash(cp:*)",
      "Bash(mv:*)",
      "Bash(touch:*)",
      "Bash(chmod:*)",
      "Bash(diff:*)",
      "Bash(sort:*)",
      "Bash(uniq:*)",
      "Bash(xargs:*)",
      "Bash(tr:*)",
      "Bash(cut:*)",
      "Bash(tee:*)",
      "Bash(date:*)",
      "Bash(pwd:*)",
      "Bash(basename:*)",
      "Bash(dirname:*)",
      "Bash(realpath:*)",
      "Bash(env:*)",
      "Bash(export:*)"
    ],
    "deny": [
      "Bash(rm -rf /:*)",
      "Bash(rm -rf ~:*)",
      "Bash(rm -rf /*:*)",
      "Bash(sudo:*)",
      "Bash(curl * | sh:*)",
      "Bash(wget * | sh:*)",
      "Edit(//.env)",
      "Edit(//.*credentials*)",
      "Edit(//.ssh/**)"
    ]
  },
  "sandbox": {
    "filesystem": {
      "allowWrite": ["."],
      "denyWrite": ["/.git"]
    }
  }
}
```

### Sandbox Template (`sandbox/default.json`)

Default sandbox configuration applied when sandboxing is enabled.

```json
{
  "enabled": true,
  "filesystem": {
    "allowWrite": ["."],
    "denyWrite": ["/.git"]
  },
  "network": {
    "allowedDomains": []
  }
}
```

Network domains are populated additively from stack and rule modules.

---

## Skills

### `/guardrail init`

**Purpose:** Detect project tech stacks, present a configuration menu, and generate a tuned `settings.local.json` with permission allowlists and optional sandbox configuration.

**Phases:**

#### Phase 1: Stack Detection

1. Run `scripts/detect-stack.sh` or equivalent logic to scan the current project directory.
2. For each stack module in `stacks/`, evaluate `detect.files` patterns against the project.
3. Produce a list of detected stacks with confidence indicators.

**Detection heuristics by stack:**

| Stack | Detection Files |
|-------|----------------|
| Python | `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile`, `*.py` |
| Node | `package.json`, `node_modules/`, `*.js`, `*.ts`, `tsconfig.json` |
| Rust | `Cargo.toml`, `*.rs` |
| Go | `go.mod`, `go.sum`, `*.go` |
| Shell | `*.sh`, `Makefile` |

#### Phase 2: Interactive Configuration Menu

Present the user with:

1. **Detected stacks** — pre-selected, user can deselect or add others
2. **Available rule modules** — `git`, `web`, `docker`, MCP integrations, etc.
3. **Sandbox toggle** — enable/disable sandboxing with preview of config
4. **Preview** — show the complete merged `settings.local.json` before writing

Menu format (rendered in conversation):
```
Detected stacks:
  [x] python (pyproject.toml, *.py found)
  [ ] node
  [ ] rust
  [ ] go
  [x] shell (Makefile found)

Rule modules:
  [x] git (recommended)
  [x] web (recommended)
  [ ] docker
  [ ] mcp-ideate
  [ ] mcp-cyberbrain

Sandbox: [x] enabled (default template)
```

User can toggle selections, then confirm to proceed.

#### Phase 3: Merge and Write

1. Start with `_base.json` permissions.
2. Merge in each selected stack module's permissions and sandbox config.
3. Merge in each selected rule module's permissions.
4. If sandbox enabled, merge sandbox configs from all modules into the sandbox template.
5. Deduplicate and sort the final permission arrays.
6. Write to `.claude/settings.local.json` in the target project.

**Merge rules:**
- `allow` arrays are unioned (deduplicated).
- `deny` arrays are unioned (deduplicated).
- `deny` rules always take precedence over `allow` at evaluation time (this is Claude Code's built-in behavior).
- Sandbox `allowedDomains` are unioned across all modules.
- Sandbox `filesystem` rules use most-restrictive-wins for `denyWrite`, union for `allowWrite`.

#### Phase 4: Confirmation

Show the final configuration and ask for explicit confirmation before writing.

**Existing settings handling:**
- If `.claude/settings.local.json` already exists, show a diff of what would change.
- Offer to merge (add new rules to existing) or replace (overwrite with generated config).
- Never silently overwrite existing configuration.

---

### `/guardrail audit`

**Purpose:** Analyze an existing project's `settings.local.json`, identify rules that can be generalized or removed, and offer to clean up.

**Phases:**

#### Phase 1: Load and Classify

1. Read the project's `.claude/settings.local.json`.
2. Classify each permission rule into categories:
   - **Generalizable** — A specific rule that matches a known pattern (e.g., `Bash(git status:*)` → already covered by `Bash(git:*)`)
   - **One-off** — A rule that references specific file paths, commit hashes, or session-specific values (e.g., `Bash(for id in af2eb7f6bb2e323aa...)`)
   - **Redundant** — A rule already covered by a more general rule in the same file or in global settings
   - **Risky** — A rule that allows potentially dangerous operations (e.g., `Bash(rm -rf:*)`, `Bash(sudo:*)`)
   - **Clean** — A well-formed, appropriately scoped rule

#### Phase 2: Report

Present findings grouped by category:

```
Audit of .claude/settings.local.json (47 rules)

Generalizable (12 rules):
  Bash(git status:*) → covered by Bash(git:*)
  Bash(git add:*)    → covered by Bash(git:*)
  Bash(git push:*)   → covered by Bash(git:*)
  ...

One-off / Session Artifacts (18 rules):
  Bash(for id in af2eb7f6bb2e323aa a68747af77c8dc099) — appears session-specific
  Bash(do echo "=== $id ===") — loop fragment
  ...

Redundant (5 rules):
  Bash(python3:*) — already in global settings
  ...

Risky (1 rule):
  Bash(rm -rf:*) — allows recursive deletion anywhere

Clean (11 rules):
  Bash(pytest:*) — appropriately scoped
  ...
```

#### Phase 3: Fix Options

After presenting the report, offer three modes:

1. **Fix all** — Apply all recommended changes at once
2. **Fix one-by-one** — Step through each recommendation, confirm or skip individually
3. **Export only** — Write the audit report to a file without modifying settings

**Fix actions:**
- **Generalizable** → Replace with the generalized form
- **One-off** → Remove (with confirmation)
- **Redundant** → Remove
- **Risky** → Flag for user decision (keep, modify, or remove)
- **Clean** → Keep as-is

After fixes, show the cleaned configuration and confirm before writing.

---

## Stack Module Reference

### `_base.json` — Universal Defaults

Always included. Allows read-only and common non-destructive CLI tools. Denies known-dangerous patterns. Sets up default sandbox filesystem rules.

### `python.json` — Python Stack

**Detects:** `requirements.txt`, `pyproject.toml`, `setup.py`, `Pipfile`, `*.py`

**Allows:** `python3`, `python`, `pip3`, `pip`, `pytest`, `mypy`, `ruff`, `black`, `isort`, `poetry`, `pdm`, `uv`, `tox`, `nox`

**Sandbox network:** `pypi.org`, `files.pythonhosted.org`

### `node.json` — Node.js/TypeScript Stack

**Detects:** `package.json`, `tsconfig.json`, `*.js`, `*.ts`

**Allows:** `node`, `npm`, `npx`, `yarn`, `pnpm`, `jest`, `vitest`, `eslint`, `prettier`, `tsc`, `tsx`, `next`, `vite`

**Sandbox network:** `registry.npmjs.org`, `registry.yarnpkg.com`

### `rust.json` — Rust Stack

**Detects:** `Cargo.toml`, `*.rs`

**Allows:** `cargo`, `rustc`, `rustup`, `clippy`, `rustfmt`

**Sandbox network:** `crates.io`, `static.crates.io`

### `go.json` — Go Stack

**Detects:** `go.mod`, `go.sum`, `*.go`

**Allows:** `go`, `gofmt`, `golint`, `golangci-lint`

**Sandbox network:** `proxy.golang.org`, `sum.golang.org`

### `shell.json` — Shell/Make Stack

**Detects:** `Makefile`, `*.sh`, `justfile`

**Allows:** `make`, `just`, `bash`, `sh`, `shellcheck`

---

## Rule Module Reference

### `git.json` — Git Operations

**Allows:** `Bash(git:*)`
**Denies:** `Bash(git push --force:*)`, `Bash(git reset --hard:*)`

### `web.json` — Web Research

**Allows:** `WebSearch`, `WebFetch` for common documentation domains

### `docker.json` — Docker/Container Operations

**Allows:** `Bash(docker:*)`, `Bash(docker-compose:*)`, `Bash(docker compose:*)`
**Denies:** `Bash(docker run --privileged:*)`

### `mcp-ideate.json` — Ideate Plugin MCP Tools

**Allows:** `mcp__plugin_ideate_ideate-artifact-server__*`
**Auto:** true (if ideate plugin is detected as enabled)

### `mcp-cyberbrain.json` — Cyberbrain Plugin MCP Tools

**Allows:** `mcp__plugin_cyberbrain_cyberbrain__*`
**Auto:** true (if cyberbrain plugin is detected as enabled)

---

## Configuration Merging Algorithm

```
1. Start with empty allow[], deny[], sandbox{}
2. Load _base.json → merge into result
3. For each selected stack:
   a. Load stacks/{stack}.json
   b. Union allow[] arrays (deduplicate)
   c. Union deny[] arrays (deduplicate)
   d. Merge sandbox.network.allowedDomains (union)
   e. Merge sandbox.filesystem (union allowWrite, union denyWrite)
4. For each selected rule module:
   a. Load rules/{rule}.json
   b. Same merge logic as stacks
5. Sort allow[] and deny[] alphabetically
6. Assemble final settings.local.json:
   {
     "permissions": {
       "allow": [...merged allow...],
       "deny": [...merged deny...]
     },
     "sandbox": { ...merged sandbox... }
   }
```

---

## Edge Cases and Considerations

### Existing settings.local.json

- `init` must handle projects that already have settings.
- Default behavior: show diff, offer merge or replace.
- Merge mode: add new rules, don't remove existing ones.
- Replace mode: back up existing file to `.claude/settings.local.json.bak` before overwriting.

### Global vs Local Settings Interaction

- Guardrail v1 only writes project-level `settings.local.json`.
- Audit should be aware of global settings (`~/.claude/settings.json`) to identify redundant local rules.
- Rules in global settings don't need to be repeated locally.

### Stack Detection Ambiguity

- A project may match multiple stacks (e.g., Python + Shell + Docker).
- All detected stacks are pre-selected; user can deselect.
- If no stacks are detected, only `_base` rules apply.

### Custom Stacks and Rules

- Users can add custom JSON files to `stacks/` or `rules/` following the schema.
- Custom files are discovered automatically — no registration needed.
- File naming convention: lowercase, hyphenated (e.g., `my-custom-stack.json`).

### Deny Rule Semantics

- Claude Code evaluates deny rules before allow rules.
- A deny rule cannot be overridden by an allow rule.
- Guardrail's deny rules are conservative safety nets, not user-facing restrictions.

---

## Future Work

### `/guardrail policy` (v2)

Hook-based policy engine that provides context-aware permission decisions:
- Path-based rules (allow within project, deny outside)
- Command classification (read-only vs. mutating)
- Configurable escalation policies
- Audit logging

### Global Settings Management

- Option to write generalized rules to global `~/.claude/settings.json` instead of per-project.
- Cross-project audit: scan all projects for common patterns to promote to global.

### Permission Analytics

- Track which rules are actually being matched.
- Identify unused rules that can be removed.
- Suggest new rules based on frequently prompted commands.

---

## Success Criteria

1. Running `/guardrail init` on any project in `~/code/` produces a working `settings.local.json` that eliminates 80%+ of permission prompts for typical development tasks.
2. Running `/guardrail audit` on existing projects (beepboop, ideate, outpost, etc.) correctly identifies generalizable, one-off, and redundant rules.
3. Stack detection correctly identifies Python, Node, Rust, Go, and Shell projects.
4. No security regression — destructive operations (`rm -rf /`, `sudo`, force push) remain denied or prompted.
5. Custom stack/rule modules can be added by dropping a JSON file with no code changes.
