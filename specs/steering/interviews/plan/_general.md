# Interview — General (Cross-Domain)

<!-- domains: permissions, sandbox, plugin-system -->
**Q: Stack detection mechanism — shell script or inline?**
A: Inline detection using Glob against JSON-defined patterns. No shell script. JSON files are the single source of truth.

<!-- domains: plugin-system -->
**Q: Plugin distribution?**
A: Add to claude-marketplace registry from the start.

<!-- domains: permissions, sandbox -->
**Q: Testing strategy?**
A: Test by running init on existing projects and comparing output against accumulated settings.local.json files.

<!-- domains: permissions, plugin-system -->
**Q: README?**
A: Include a README highlighting usage, key knowledge users should have, and important details.
