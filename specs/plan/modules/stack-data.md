# Module: stack-data

## Scope

Owns all files in `stacks/` — the JSON data files that define tech stack detection patterns, permission rules, and sandbox configurations.

Files:
- `stacks/_base.json` — Universal safe defaults, always included
- `stacks/python.json` — Python development stack
- `stacks/node.json` — Node.js/TypeScript stack
- `stacks/rust.json` — Rust stack
- `stacks/go.json` — Go stack
- `stacks/shell.json` — Shell/Make stack

NOT responsible for: rule modules (`rules/`), sandbox template (`sandbox/`), skill logic, plugin registration.

## Provides

- **StackModule JSON files** — Each file conforms to the StackModule schema and provides:
  - `name` and `description` for display
  - `detect.files` glob patterns and `detect.match` mode for stack detection
  - `permissions.allow` and optional `permissions.deny` arrays
  - Optional `sandbox.filesystem` and `sandbox.network` overrides

## Requires

Nothing. Stack data files are leaf nodes with no dependencies on other modules.

## Boundary Rules

- Every stack JSON file must conform to the StackModule schema exactly.
- `_base.json` must always be present. It is the foundation that other stacks build on.
- `_base.json` must include the safety deny list (rm -rf, sudo, curl|sh, etc.).
- Detection patterns must use glob syntax compatible with Claude Code's Glob tool.
- Permission rule strings must use Claude Code's permission syntax (`Bash(command:*)`, `Read`, `WebFetch(domain:x)`, etc.).
- Stack files must not reference other stack files. Each is self-contained.
- No executable logic — these are pure data files.

## Internal Design Notes

### _base.json
Always included regardless of detection. Contains:
- Common read-only CLI tools (ls, cat, head, tail, grep, find, etc.)
- Common non-destructive file ops (mkdir, cp, mv, touch, chmod)
- Utility commands (date, pwd, basename, dirname, realpath, env)
- Safety deny list: rm -rf /, rm -rf ~, sudo, curl|sh, wget|sh, .env edits, credential file edits, .ssh edits
- Default sandbox filesystem: allowWrite ["."], denyWrite ["/.git"]

### Stack files (python, node, rust, go, shell)
Each provides:
- Detection patterns specific to that ecosystem's marker files
- Allow rules for that ecosystem's CLI tools (interpreters, package managers, linters, formatters, test runners)
- Sandbox network domains for that ecosystem's package registries

### Extensibility
Users add custom stacks by placing JSON files in `.claude/guardrail/stacks/`. A file with the same name as a plugin-scoped stack replaces it entirely. New filenames are additive.
