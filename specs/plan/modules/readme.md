# Module: readme

## Scope

Owns `README.md` — the user-facing documentation for the guardrail plugin.

Responsible for:
- Explaining what guardrail does and why
- Usage instructions for both skills (`/guardrail init` and `/guardrail audit`)
- Explaining the stack/rule data model and how to extend it
- Documenting the project override mechanism (`.claude/guardrail/`)
- Listing included stacks and rule modules
- Installation instructions (marketplace and manual)

NOT responsible for: skill behavior (that is defined in the skill files), data file content, plugin registration.

## Provides

- **README.md** — Human-readable documentation. No other module depends on this file.

## Requires

- Finalized skill names and descriptions (from: `init-skill`, `audit-skill`) — to document usage accurately.
- Finalized stack list (from: `stack-data`) — to list included stacks.
- Finalized rule list (from: `rule-data`) — to list included rule modules.
- JSON schemas (from: architecture contracts) — to document the extension format.

## Boundary Rules

- Must not define behavior. The README describes what the skills do; it does not prescribe how.
- Must accurately reflect the current state of all other modules. If a module changes, the README must be updated.
- Must include installation instructions for both marketplace (`claude plugin add guardrail`) and manual (git clone) methods.

## Internal Design Notes

Suggested structure:
1. **Overview** — What guardrail does, the problem it solves (1-2 paragraphs)
2. **Quick Start** — Install + run init in 3 steps
3. **Skills** — `/guardrail init` and `/guardrail audit` with example interactions
4. **Included Stacks** — Table of stacks with detection files and what they allow
5. **Included Rules** — Table of rule modules with descriptions
6. **Customization** — How to add custom stacks/rules in `.claude/guardrail/`
7. **How It Works** — Brief explanation of the merge algorithm and sandbox defaults
8. **FAQ** — Common questions (e.g., "Can I use this outside a git repo?", "How do I undo?")
