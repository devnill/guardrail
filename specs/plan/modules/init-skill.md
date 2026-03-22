# Module: init-skill

## Scope

Owns `skills/init/SKILL.md` — the markdown skill file that Claude Code executes when the user invokes `/guardrail init`.

Responsible for:
- Stack detection by globbing project files against stack JSON detect patterns
- Loading and merging stack, rule, and sandbox data into a unified config
- Presenting an interactive selection menu to the user
- Detecting git repo to determine write location
- Reading existing settings.local.json and computing diffs
- Writing the merged permissions + sandbox config to settings.local.json
- Preserving non-guardrail keys in existing settings files

NOT responsible for: the content of stack/rule/sandbox JSON files, audit logic, plugin registration.

## Provides

- **`/guardrail init` skill** — A complete markdown instruction set that Claude follows to perform the init workflow. This is the skill file, not an API.

## Requires

- **StackModule JSON files** (from: `stack-data`) — Read at runtime from `${CLAUDE_PLUGIN_ROOT}/stacks/*.json` and `.claude/guardrail/stacks/*.json`. Must conform to the StackModule schema.
- **RuleModule JSON files** (from: `rule-data`) — Read at runtime from `${CLAUDE_PLUGIN_ROOT}/rules/*.json` and `.claude/guardrail/rules/*.json`. Must conform to the RuleModule schema.
- **SandboxTemplate JSON** (from: `sandbox-data`) — Read at runtime from `${CLAUDE_PLUGIN_ROOT}/sandbox/default.json`. Must conform to the SandboxTemplate schema.

## Boundary Rules

- Must not hardcode permission rules, stack names, or detection patterns. All domain knowledge comes from JSON data files.
- Must not write to any settings key other than `permissions` and `sandbox`. All other keys in settings.local.json are preserved exactly.
- Must not silently overwrite existing configuration. If settings.local.json exists, must show a diff and ask before writing.
- Must not write to global `~/.claude/settings.json`.
- Must not execute external scripts or binaries. Stack detection uses Claude Code's built-in Glob tool.
- Project-scoped override files with the same name as a plugin-scoped file replace the plugin version entirely (no deep merge of same-name files).

## Internal Design Notes

The skill is structured in four phases as natural-language instructions:

### Phase 1: Discovery
- Use Glob to find all `*.json` in `${CLAUDE_PLUGIN_ROOT}/stacks/`
- Use Glob to find all `*.json` in `.claude/guardrail/stacks/` (project overrides)
- Same-name project files replace plugin files
- Read each stack JSON, parse detect.files patterns
- For each stack, Glob the project directory for detect.files matches
- Build list: detected stacks (matched) and available stacks (not matched)
- Read all rule module JSONs from both locations

### Phase 2: Interactive Menu
- Present detected stacks as pre-selected
- Present undetected stacks as available but unselected
- Present rule modules: those with `auto: true` are pre-selected
- Present sandbox toggle (default: enabled)
- User confirms or adjusts

### Phase 3: Merge
- Start with `_base.json` (always included, even if not explicitly selected)
- For each selected stack: union allow[], union deny[], merge sandbox fields
- For each selected rule: union allow[], union deny[]
- If sandbox enabled: start from sandbox/default.json, merge in stack sandbox configs
- Deduplicate and sort all arrays alphabetically

### Phase 4: Write
- Check for git repo: `git rev-parse --git-dir`
  - In git repo: target is `{git-root}/.claude/settings.local.json`
  - Not in git repo: confirm location with user (default: `{cwd}/.claude/settings.local.json`)
- If target file exists:
  - Read it, parse JSON
  - Show diff of what would change (permissions + sandbox keys only)
  - Offer: merge (add new rules to existing) or replace (overwrite permissions + sandbox)
  - If replace: back up to `settings.local.json.bak`
- If target file does not exist:
  - Create `.claude/` directory if needed
  - Write full settings object
- Show final config, get explicit confirmation before writing
