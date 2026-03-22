# Module: rule-data

## Scope

Owns all files in `rules/` — the JSON data files that define cross-cutting (stack-independent) permission modules.

Files:
- `rules/git.json` — Git version control operations
- `rules/web.json` — Web search and documentation fetching
- `rules/docker.json` — Docker/container operations
- `rules/mcp-ideate.json` — Ideate plugin MCP tool permissions
- `rules/mcp-cyberbrain.json` — Cyberbrain plugin MCP tool permissions

NOT responsible for: stack definitions (`stacks/`), sandbox template, skill logic, plugin registration.

## Provides

- **RuleModule JSON files** — Each file conforms to the RuleModule schema and provides:
  - `name` and `description` for display
  - Optional `auto` flag (if true, always included without user prompt)
  - `permissions.allow` and optional `permissions.deny` arrays

## Requires

Nothing. Rule data files are leaf nodes with no dependencies on other modules.

## Boundary Rules

- Every rule JSON file must conform to the RuleModule schema exactly.
- Rule files must not include `detect` blocks — they are not auto-detected from project files. Selection is via the interactive menu or the `auto` flag.
- Permission rule strings must use Claude Code's permission syntax.
- Deny rules in rule modules are safety-critical (e.g., git push --force, docker run --privileged). They must not be removed without understanding the security implications.
- Rule files must not reference other rule files. Each is self-contained.
- No executable logic — these are pure data files.

## Internal Design Notes

### git.json
- Allows: `Bash(git:*)`
- Denies: `Bash(git push --force:*)`, `Bash(git reset --hard:*)`

### web.json
- Allows: `WebSearch`, `WebFetch` for common documentation domains (docs.anthropic.com, github.com, stackoverflow.com)

### docker.json
- Allows: `Bash(docker:*)`, `Bash(docker-compose:*)`, `Bash(docker compose:*)`
- Denies: `Bash(docker run --privileged:*)`

### mcp-ideate.json and mcp-cyberbrain.json
- Allow MCP tool patterns for these specific plugins
- `auto: true` — included without prompting since they only matter if the corresponding plugin is installed

### Extensibility
Users add custom rules by placing JSON files in `.claude/guardrail/rules/`. Same-name files replace plugin versions. New filenames are additive.
