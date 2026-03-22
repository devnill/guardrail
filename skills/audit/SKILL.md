---
description: "Analyze existing permission rules, identify consolidations, risks, and cruft"
user-invocable: true
argument-hint: ""
---

# Guardrail Audit

You are performing a permission audit of Claude Code settings. Follow these three phases exactly.

## Phase 1: Load and Classify

### Step 1: Load settings files

1. Read `~/.claude/settings.json`. Extract `permissions.allow` and `permissions.deny` arrays. If the file does not exist, note "No global settings file found" and skip global analysis.
2. Read `.claude/settings.local.json` in the current project. Extract `permissions.allow` and `permissions.deny` arrays. If this file does not exist, report: **"No .claude/settings.local.json found. Run `/guardrail init` first."** and stop.
3. If both permission arrays in local settings are empty or missing, report: **"No permission rules configured."** and stop.

### Step 2: Load reference patterns

Read all JSON files from:
- `${CLAUDE_PLUGIN_ROOT}/stacks/` — stack definitions
- `${CLAUDE_PLUGIN_ROOT}/rules/` — rule templates
- `.claude/guardrail/stacks/` — project-specific stack definitions (if the directory exists)
- `.claude/guardrail/rules/` — project-specific rules (if the directory exists)

From each JSON file, extract the `permissions.allow` and `permissions.deny` arrays. Combine all of these into a single reference set of known patterns. This reference set is used during classification to identify rules that match or are subsets of known stack/rule module patterns.

### Step 3: Classify each LOCAL rule

Evaluate every rule in the local settings (both allow and deny arrays). Classify each rule into exactly one of the five categories below. Apply the checks in this order — the first match wins.

#### Category 1: Risky

A rule is **risky** if any of the following apply:
- It matches patterns: `rm -rf`, `rm -r`, `sudo`, `curl * | sh`, `wget * | sh`, `--force`, `--privileged`, `--no-verify`
- It allows write access to sensitive paths: `.env`, `.ssh`, `credentials`
- It applies a very broad wildcard to a dangerous command (e.g., `Bash(rm:*)`, `Bash(curl:*)`)

#### Category 2: Redundant

A rule is **redundant** if any of the following apply:
- It is an exact duplicate of another rule in local settings
- It is an exact match of a rule already present in global settings
- It is a strict subset of a broader rule that exists in either local or global settings. For example, `Bash(git status:*)` is a strict subset of `Bash(git:*)`. To determine this: if a rule's command prefix is itself a prefix of another rule's broader wildcard pattern, the narrower rule is redundant.

#### Category 3: One-off / Session Artifacts

A rule is a **one-off** if any of the following apply:
- It contains hex strings longer than 8 characters (commit hashes, session IDs)
- It contains loop syntax fragments: `for `, `do `, `done`, `echo "=== $`
- It contains absolute paths to `/tmp/`, `/private/tmp/`, or other temp directories
- It contains absolute paths to specific files that do not exist in the current project
- It contains pipe fragments that look like partial commands pasted from a session

#### Category 4: Generalizable

A rule is **generalizable** if:
- Group all remaining unclassified local allow rules by their command prefix (the command name before the first space or `:`).
- If 3 or more rules share the same prefix AND that prefix is **not** on the safety deny list, flag the entire group as generalizable. Suggest consolidation to `Bash({prefix}:*)`.
- Also flag any individual rule that is a strict subset of a known stack/rule module pattern from the reference set — suggest adopting the broader pattern.

**Safety deny list — never suggest wildcarding these commands:**
`rm`, `sudo`, `curl`, `wget`, `chmod`, `chown`, `kill`, `pkill`, `dd`, `mkfs`, `fdisk`

#### Category 5: Clean

A rule is **clean** if none of the above categories apply. It is well-scoped, unique, and non-dangerous. No action needed.

## Phase 2: Report

Present findings using this exact structure. Include counts for each category. For every rule listed, include a brief explanation of WHY it was classified that way — do not just list rules without reasoning.

```
## Audit Report: .claude/settings.local.json

**Total rules analyzed:** {N} (local) + {N} (global, read-only)

### Risky ({N} rules)
  {rule} — {reason, e.g., "allows recursive deletion"}
  ...

### Redundant ({N} rules)
  {rule} — {reason, e.g., "already in global settings" or "subset of Bash(git:*)"}
  ...

### One-off / Session Artifacts ({N} rules)
  {rule} — {reason, e.g., "contains commit hash" or "session-specific loop"}
  ...

### Generalizable ({N} rules)
  {rule} → consolidate to {generalized form}
  {rule} → consolidate to {generalized form}
  ...

### Clean ({N} rules)
  {rule} — {reason, e.g., "appropriately scoped"}
  ...

### Global Observations (informational)
  {Any findings about global settings — risks, possible generalizations, etc.}
  Note: guardrail does not modify global settings. These are observations only.

### Summary
  Recommended removals: {N}
  Recommended consolidations: {N}
  Requires manual review: {N}
```

If all rules are clean, report a clean bill of health and skip Phase 3.

## Phase 3: Fix Options

After presenting the report, ask the user to choose one of three modes:

1. **Fix all** — Apply all recommended changes automatically, then show the resulting config and ask for confirmation before writing.
2. **Fix one-by-one** — Step through each recommendation individually. For each rule, show the rule, the recommendation, and ask the user to confirm or skip.
3. **Export only** — Write the audit report to `.claude/guardrail/audit-report.md`. Do not modify settings.

### Fix actions by category

- **Risky**: ALWAYS ask individually, even in fix-all mode. Present three options: "Keep as-is", "Modify" (let user provide new rule), or "Remove". Never auto-remove risky rules without explicit per-rule confirmation.
- **Redundant**: Remove the rule from local settings.
- **One-off / Session Artifacts**: Remove the rule from local settings.
- **Generalizable**: Replace the group of specific rules with the consolidated generalized form (e.g., replace three `Bash(npm run build:*)`, `Bash(npm test:*)`, `Bash(npm install:*)` with `Bash(npm:*)`).
- **Clean**: No action.

### Writing changes

- **NEVER modify `~/.claude/settings.json`** (global settings). All writes go to `.claude/settings.local.json` only.
- Only modify the `permissions` key. Do not touch `sandbox` or any other keys in the settings file.
- Before writing any changes, show the user the complete updated `permissions` object and require explicit confirmation (e.g., "Write these changes? [yes/no]").
- Preserve all other keys in the settings file exactly as they were.
