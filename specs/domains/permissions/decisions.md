# Decisions: Permissions

## D-1: Prefix-based generalization over pattern lookup
- **Decision**: Use prefix-based detection (shared command prefix) rather than maintaining a lookup table of command families
- **Rationale**: Prefix detection is simpler, generalizes to unknown tools, and requires no maintenance as new tools emerge
- **Source**: steering/interviews/plan/_full.md — audit classification question
- **Status**: settled

## D-2: Safety deny list — best effort for v1
- **Decision**: Ship with a best-effort deny list (rm, sudo, curl, wget, chmod, chown, kill, pkill, dd, mkfs, fdisk) and iterate based on adversarial testing
- **Rationale**: User plans to adversarially test guardrail; perfectionism here blocks shipping
- **Source**: steering/interviews/plan/_full.md — deny list question
- **Status**: settled

## D-3: Audit reads global settings but never modifies them
- **Decision**: Audit analyzes the combined effective ruleset (global + local) but only proposes fixes to local settings
- **Rationale**: Global settings affect all projects; modifying them has high blast radius and should be a deliberate user action, not an automated suggestion
- **Source**: steering/interviews/plan/_full.md — audit scope question
- **Status**: settled
