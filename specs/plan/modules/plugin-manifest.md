# Module: plugin-manifest

## Scope

Owns the plugin identity and registration files:
- `.claude-plugin/plugin.json` — plugin manifest consumed by Claude Code
- `.claude-plugin/marketplace.json` — marketplace registry entry
- `CLAUDE.md` — plugin-level instructions that Claude Code loads when the plugin is active

NOT responsible for: skill content, data file content, user documentation (README).

## Provides

- **plugin.json** — Registers the plugin name (`guardrail`), version, description, and declares two skills (`init` at `skills/init/SKILL.md`, `audit` at `skills/audit/SKILL.md`).
- **marketplace.json** — Provides the marketplace listing: name, description, author, repository URL, version, tags for discoverability.
- **CLAUDE.md** — Plugin-level context instructions. Tells Claude about guardrail's purpose, where data files live (`${CLAUDE_PLUGIN_ROOT}/stacks/`, `${CLAUDE_PLUGIN_ROOT}/rules/`, `${CLAUDE_PLUGIN_ROOT}/sandbox/`), and the project override location (`.claude/guardrail/`).

## Requires

- Finalized skill names and paths (from: `init-skill`, `audit-skill`) — needed to populate the `skills` array in plugin.json.
- Plugin version number — determined at authoring time, not derived from another module.

## Boundary Rules

- Must not contain any behavioral logic. These are declarative registration files only.
- Must follow the established Claude Code plugin structure exactly.
- `CLAUDE.md` provides orientation context only — it must not duplicate the skill instructions.
- Version in `plugin.json` and `marketplace.json` must match.

## Internal Design Notes

plugin.json structure:
```json
{
  "name": "guardrail",
  "version": "1.0.0",
  "description": "Automates permission allowlists and sandbox configuration for Claude Code projects",
  "skills": [
    {
      "name": "init",
      "path": "skills/init/SKILL.md",
      "description": "Detect project stacks and generate tuned permission + sandbox config"
    },
    {
      "name": "audit",
      "path": "skills/audit/SKILL.md",
      "description": "Analyze existing permission rules, find consolidations and risks"
    }
  ]
}
```

CLAUDE.md should mention:
- What guardrail does (one sentence)
- Where stack/rule data lives (plugin-scoped and project-scoped paths)
- That skills read JSON data files at runtime — do not hardcode permission lists
