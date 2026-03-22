# Interview Transcript — 2026-03-22

## Context
User wants to build a Claude Code plugin called "guardrail" that automates permission and sandbox configuration for projects. Initial spec provided at `specs/guardrail-v1.md`. The user has extensive experience building Claude Code plugins (beepboop, ideate, cyberbrain, hamlet) and is frustrated by constant permission prompts interrupting autonomous agent workflows.

---

**Q: Stack detection mechanism — should it be a shell script or inline?**
A: User deferred to my judgment. Decision: inline detection using Glob against JSON-defined patterns. No shell script. The JSON files are the single source of truth — the skill reads `stacks/*.json`, iterates their `detect.files` arrays, and globs for matches. Eliminates runtime dependency and maintenance of detection logic in two places.

**Q: Where does `init` write, and how does the skill locate stack/rule JSON files?**
A: `init` checks if we're in a git repo. If so, writes to project-scoped `.claude/settings.local.json`. If not in a git repo, confirms where to write with the default being the current directory. Stacks and rules load from `${CLAUDE_PLUGIN_ROOT}` (the plugin installation), but project-scoped overrides can exist in `.claude/guardrail/stacks/` and `.claude/guardrail/rules/`. This follows the cyberbrain pattern of using `.claude/` as the idiomatic location for Claude Code plugin config.

**Q: Audit skill classification logic — pattern-based or prefix-based?**
A: User is ambivalent on taxonomy. Core goal: minimize permission prompts while maintaining security against prompt injection and dangerous commands. Decision: prefix-based generalization with a safety deny list. Multiple specific rules sharing a common command prefix get consolidated. Commands where wildcarding is unsafe (rm, sudo, curl|sh) are on a deny list and never generalized.

**Q: Should `init` enable sandboxing by default or present it as opt-in?**
A: Sandboxing enabled by default. Light touch with maximum security. Sandbox is a quick win for prompt reduction.

**Q: Sandbox auto-allow mode — enabled by default?**
A: Yes, auto-allow in sandboxes by default. This is the biggest lever for reducing permission prompts.

**Q: Merge vs replace on existing settings?**
A: Never clobber without permission. If conflicts exist, ask the user. `init` only touches `permissions` and `sandbox` keys, leaves everything else untouched. Guardrail's job is to make smart suggestions to maximize productivity and safety, especially when current settings could be improved.

**Q: Audit scope — local only or global + local?**
A: Both. Audit reads global `~/.claude/settings.json` and local `settings.local.json` as a combined effective ruleset. Reports risks and optimizations that affect the current project, inherited or otherwise. Proposed fixes specify which file they'd modify and require explicit confirmation.

**Q: Plugin distribution?**
A: Add to claude-marketplace registry from the start. Include marketplace registration in plan scope.

**Q: Deny list contents?**
A: Best effort for v1. Will iterate later if the project shows promise.

**Q: Testing strategy?**
A: Test by running `init` on existing projects and comparing output against accumulated `settings.local.json` files. Manual validation.

**Q: README?**
A: Yes, include a README highlighting usage, key knowledge users should have, and important details.
