# Questions: Permissions

## Q-1: Should audit eventually propose global settings changes?
- **Question**: V1 treats global settings as read-only for audit. Should v2 allow audit to propose changes to global settings (with appropriate confirmation)?
- **Source**: steering/interviews/plan/_full.md — audit scope discussion
- **Impact**: Users with risky global rules get informed but can't fix via guardrail
- **Status**: open
- **Reexamination trigger**: User feedback requesting global audit fixes

## Q-2: How should compound commands be classified?
- **Question**: When a rule like `Bash(git status && npm test)` exists, should audit classify it as one-off, generalizable to both `git` and `npm`, or leave it as-is?
- **Source**: architecture design — edge case
- **Impact**: Affects classification accuracy for rules that combine multiple tools
- **Status**: open
- **Reexamination trigger**: Encountering compound command rules in real audit runs
