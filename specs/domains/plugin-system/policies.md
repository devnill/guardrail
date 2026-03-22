# Policies: Plugin System

## P-1: Same-name project overrides replace entirely
When a project has `.claude/guardrail/stacks/python.json` and the plugin ships `stacks/python.json`, the project version replaces the plugin version entirely. No deep merge of same-name files.
- **Derived from**: GP-6 (Composable and Extensible) — simplest merge model
- **Established**: planning phase
- **Status**: active

## P-2: Data files are the single source of domain knowledge
Skills (markdown) contain workflow logic. JSON files contain domain knowledge (detection patterns, permissions, deny lists). Neither duplicates the other's concerns.
- **Derived from**: GP-2 (Data-Driven, Not Code-Driven)
- **Established**: planning phase
- **Status**: active

## P-3: Discovery uses ${CLAUDE_PLUGIN_ROOT} for plugin files
All skill references to plugin-scoped data files use `${CLAUDE_PLUGIN_ROOT}` to ensure correct resolution regardless of where the plugin is installed.
- **Derived from**: Constraint 1 (Claude Code plugin format)
- **Established**: planning phase
- **Status**: active
