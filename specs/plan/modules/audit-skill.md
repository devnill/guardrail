# Module: audit-skill

## Scope

Owns `skills/audit/SKILL.md` — the markdown skill file that Claude Code executes when the user invokes `/guardrail audit`.

Responsible for:
- Reading global settings (`~/.claude/settings.json`) and local settings (`.claude/settings.local.json`)
- Building the combined effective ruleset
- Classifying each local permission rule into categories (generalizable, one-off, redundant, risky, clean)
- Presenting a categorized audit report
- Offering fix modes (fix-all, fix-one-by-one, export-only)
- Applying fixes to local settings with user confirmation

NOT responsible for: stack/rule JSON content, init workflow, plugin registration, modifying global settings.

## Provides

- **`/guardrail audit` skill** — A complete markdown instruction set that Claude follows to perform the audit workflow.

## Requires

- **StackModule JSON files** (from: `stack-data`) — Read at runtime to identify known command patterns for generalization matching.
- **RuleModule JSON files** (from: `rule-data`) — Read at runtime to identify known command patterns for generalization matching.

## Boundary Rules

- Must not modify global `~/.claude/settings.json`. Global settings are read-only for analysis.
- Must not write to any settings key other than `permissions`. Sandbox audit is informational only in v1.
- Must not silently remove or modify rules. All changes require explicit user confirmation.
- Must not hardcode classification patterns. Uses prefix-based generalization with a safety deny list loaded from data files.
- Must present global findings in a separate clearly-marked informational section.

## Internal Design Notes

The skill is structured in three phases:

### Phase 1: Load and Classify

**Loading:**
- Read `~/.claude/settings.json` -> extract `permissions.allow` and `permissions.deny`
- Read `.claude/settings.local.json` -> extract `permissions.allow` and `permissions.deny`
- Read stack and rule JSONs to build a reference set of known patterns

**Classification of each rule in local settings.allow[]:**

| Category | Detection Heuristic |
|----------|-------------------|
| Generalizable | A specific command (e.g. `Bash(git status:*)`) where a broader prefix exists in a known stack/rule module (e.g. `Bash(git:*)`). Multiple rules sharing a command prefix. |
| One-off | Contains hex strings (commit hashes), loop fragments (`for`, `do`, `done`), absolute paths to temp files, session-specific content. |
| Redundant | Already covered by a broader rule in the same local file, or already present in global settings. |
| Risky | Matches known dangerous patterns: `rm -rf`, `sudo`, `curl|sh`, `wget|sh`, `--force`, `--privileged`. Uses the deny patterns from `_base.json`. |
| Clean | None of the above. Well-scoped, unique, non-dangerous. |

**Prefix-based generalization:**
- Group rules by command prefix (the part before the first space or `:`)
- If 3+ rules share a prefix and the prefix is not on the safety deny list, suggest consolidation to `Bash({prefix}:*)`
- Safety deny list (never generalize): `rm`, `sudo`, `curl`, `wget`, `chmod`, `chown`, `kill`, `pkill`

### Phase 2: Report

Present findings grouped by category with counts:
- Total rules analyzed
- Per-category breakdown with specific examples
- Global observations section (informational, not actionable)
- Summary of recommended actions

### Phase 3: Fix Options

Three modes offered to user:
1. **Fix all** — Apply all recommendations at once, show result, confirm before write
2. **Fix one-by-one** — Step through each recommendation individually, user confirms or skips
3. **Export only** — Write audit report to a file (e.g., `.claude/guardrail/audit-report.md`), no settings changes

Fix actions by category:
- Generalizable -> replace specific rules with generalized form
- One-off -> remove (with individual confirmation in one-by-one mode)
- Redundant -> remove
- Risky -> present to user: keep, modify, or remove (always individual confirmation)
- Clean -> keep as-is

After fixes, show final cleaned config and confirm before writing.
