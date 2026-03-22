# Domain Registry

current_cycle: 0

## Domains

### permissions
Covers the permission rule system: allow/deny syntax, wildcard semantics, rule generalization, classification heuristics, and the deny-always-wins evaluation model.
Files: domains/permissions/policies.md, decisions.md, questions.md

### sandbox
Covers OS-level sandboxing: Seatbelt/bubblewrap configuration, auto-allow mode, filesystem and network restrictions, platform support.
Files: domains/sandbox/policies.md, decisions.md, questions.md

### plugin-system
Covers Claude Code plugin mechanics: manifest format, skill structure, marketplace registration, project-scoped overrides, data file discovery.
Files: domains/plugin-system/policies.md, decisions.md, questions.md

## Cross-Cutting Concerns
Security model spans both permissions and sandbox domains — changes in one affect the other's effectiveness.
