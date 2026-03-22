# Interview — Permissions Domain

<!-- domains: permissions -->
**Q: Audit skill classification logic — pattern-based or prefix-based?**
A: Core goal: minimize permission prompts while maintaining security against prompt injection and dangerous commands. Decision: prefix-based generalization with a safety deny list. Multiple specific rules sharing a common command prefix get consolidated. Commands where wildcarding is unsafe (rm, sudo, curl|sh) are on a deny list and never generalized.

<!-- domains: permissions -->
**Q: Merge vs replace on existing settings?**
A: Never clobber without permission. If conflicts exist, ask the user. Init only touches permissions and sandbox keys, leaves everything else untouched. Guardrail's job is to make smart suggestions to maximize productivity and safety, especially when current settings could be improved.

<!-- domains: permissions -->
**Q: Audit scope — local only or global + local?**
A: Both. Audit reads global and local as a combined effective ruleset. Reports risks and optimizations that affect the current project. Proposed fixes specify which file they'd modify and require explicit confirmation.

<!-- domains: permissions -->
**Q: Deny list contents?**
A: Best effort for v1. Will iterate later if the project shows promise.
