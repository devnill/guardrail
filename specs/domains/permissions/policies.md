# Policies: Permissions

## P-1: Generalize by prefix, never by command family assumption
When consolidating rules, use prefix-based detection (3+ rules sharing a command prefix → suggest `Bash({prefix}:*)`). Do not maintain a hardcoded lookup of "git subcommands" or similar — the prefix is sufficient and generalizes to unknown tools.
- **Derived from**: GP-4 (Smart Suggestions Over Dumb Lists)
- **Established**: planning phase
- **Status**: active

## P-2: Safety deny list commands are never wildcarded
Commands on the safety deny list (rm, sudo, curl, wget, chmod, chown, kill, pkill, dd, mkfs, fdisk) must never be suggested for generalization to `Bash({cmd}:*)`. Individual specific rules for these commands may exist but require manual review.
- **Derived from**: GP-1 (Maximize Autonomy Within Safety Boundaries)
- **Established**: planning phase
- **Status**: active

## P-3: Deny rules are the immutable safety net
Claude Code evaluates deny before allow. Guardrail's deny rules are not user-convenience features — they are security controls. Deny rules should be conservative and rarely removed.
- **Derived from**: GP-5 (Defense in Depth)
- **Established**: planning phase
- **Status**: active

## P-4: Only touch permissions and sandbox keys
When writing to settings.local.json, guardrail reads the full file, modifies only `permissions` and `sandbox`, and writes back with all other keys preserved unchanged.
- **Derived from**: GP-3 (Never Clobber Without Permission)
- **Established**: planning phase
- **Status**: active
