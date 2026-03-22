# Research: Plugin Project-Scoped Configuration Patterns

## Date: 2026-03-22

## Question
What is the idiomatic pattern for Claude Code plugins to store project-scoped configuration?

## Findings

### Existing Patterns in Dan's Plugins

| Plugin | Project Config Location | Pattern |
|--------|------------------------|---------|
| Hamlet | `.hamlet/config.json` | Dedicated hidden directory |
| Cyberbrain | `.claude/cyberbrain.local.json` | Inside `.claude/` directory |
| Ideate | `.ideate.json` | Root-level dotfile |
| Beepboop | N/A | No project-scoped config |

### Discovery Mechanism
All three plugins implementing project config use **upward directory tree walking** — start at cwd, check for config file, walk to parent, repeat until found or hitting a boundary (home dir for cyberbrain, filesystem root for ideate).

### Key Observations
1. `.claude/` is the idiomatic Claude Code convention — it's where `settings.json` and `settings.local.json` already live.
2. Cyberbrain follows this pattern with `.claude/cyberbrain.local.json`.
3. Hamlet uses a standalone directory (`.hamlet/`) because it's also a standalone CLI tool, not just a Claude Code plugin.
4. All plugins support explicit argument override that takes precedence over file discovery.
5. Project configs are intentionally minimal — small JSON files with a few fields.

### Recommendation for Guardrail
Use `.claude/guardrail/` as the project-scoped override directory. This:
- Follows the `.claude/` convention (idiomatic)
- Needs a directory (not a single file) because users may have multiple custom stack/rule JSON files
- Supports `stacks/` and `rules/` subdirectories mirroring the plugin structure
- Leaves room for a future `config.json` for project-level settings
