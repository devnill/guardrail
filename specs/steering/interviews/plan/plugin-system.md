# Interview — Plugin System Domain

<!-- domains: plugin-system -->
**Q: Where does init write, and how does the skill locate stack/rule JSON files?**
A: Init checks if we're in a git repo. If so, writes to project-scoped .claude/settings.local.json. If not in a git repo, confirms where to write with the default being the current directory. Stacks and rules load from ${CLAUDE_PLUGIN_ROOT} (the plugin installation), but project-scoped overrides can exist in .claude/guardrail/stacks/ and .claude/guardrail/rules/. This follows the cyberbrain pattern of using .claude/ as the idiomatic location.
