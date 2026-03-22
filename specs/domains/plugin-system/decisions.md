# Decisions: Plugin System

## D-1: Project overrides in .claude/guardrail/
- **Decision**: Project-scoped custom stacks and rules live in `.claude/guardrail/stacks/` and `.claude/guardrail/rules/`
- **Rationale**: Follows the `.claude/` convention (idiomatic for Claude Code), needs a directory not a file (multiple custom modules), mirrors the plugin structure
- **Assumes**: Upward directory tree walking not needed for v1 — project root is determined by git repo detection
- **Source**: steering/research/plugin-config-patterns.md
- **Status**: settled

## D-2: Distributed via claude-marketplace
- **Decision**: Plugin registered in Dan's claude-marketplace registry from day one
- **Rationale**: Consistent with other plugins (beepboop, ideate, cyberbrain, hamlet)
- **Source**: steering/interviews/plan/_full.md — plugin distribution question
- **Status**: settled
