# Execution Strategy

## Mode
Batched parallel

## Parallelism
Max concurrent agents: 4

## Worktrees
Enabled: no
Reason: This is a new project with no existing code. No risk of conflicting with in-progress work. Files are small and independent enough that worktree isolation adds overhead without benefit.

## Review Cadence
After every batch (group)

## Work Item Groups

Group 1 (parallel): 001, 002, 003, 004, 005, 006
- All data files and plugin manifest. No dependencies between them.
- Max parallelism: 6 items

Group 2 (parallel, depends on Group 1): 007, 008
- Both skills. Depend on data files being finalized for schema reference.
- Can run in parallel since they don't share files.

Group 3 (sequential, depends on Group 2): 009
- README. Depends on both skills being finalized to document accurately.

Group 4 (sequential, depends on Group 3): 010
- Marketplace registration. Depends on everything being finalized.
- Involves a separate repository (claude-marketplace).

## Dependency Graph

```
001 (manifest) ──────────────────────────────┐
002 (_base.json) ───┬───────────────────┐     │
003 (python.json) ──┤                   │     │
004 (node/rust/go/shell) ──┤            │     │
005 (rule modules) ─┤                   │     │
006 (sandbox template) ──┘              │     │
                    │                   │     │
              ┌─────▼──────┐    ┌───────▼─┐   │
              │ 007 (init) │    │008(audit)│   │
              └─────┬──────┘    └────┬────┘   │
                    │                │        │
                    └────────┬───────┘        │
                             │                │
                      ┌──────▼──────┐         │
                      │ 009 (README)│         │
                      └──────┬──────┘         │
                             │                │
                      ┌──────▼────────────────▼┐
                      │ 010 (marketplace reg)  │
                      └────────────────────────┘
```

## Agent Configuration
Model for workers: sonnet
Model for reviewers: opus
Permission mode: acceptEdits
