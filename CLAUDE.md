# Guardrail

Automates permission allowlists and sandbox configuration for Claude Code projects.

## Data file locations

- `${CLAUDE_PLUGIN_ROOT}/stacks/` — stack detection definitions
- `${CLAUDE_PLUGIN_ROOT}/rules/` — permission rule templates
- `${CLAUDE_PLUGIN_ROOT}/sandbox/` — sandbox configuration templates

## Project overrides

- `.claude/guardrail/stacks/` — project-specific stack definitions
- `.claude/guardrail/rules/` — project-specific permission rules

## Note

Skills read JSON data files at runtime — never hardcode permission lists in skill markdown.
