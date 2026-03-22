# Module: sandbox-data

## Scope

Owns `sandbox/default.json` — the default sandbox configuration template that init uses as the starting point for sandbox settings.

NOT responsible for: per-stack sandbox overrides (those live in stack JSON files in `stack-data`), sandbox logic or merging (that is `init-skill`'s job), rule modules, plugin registration.

## Provides

- **SandboxTemplate JSON** — A single file conforming to the SandboxTemplate schema:
  - `enabled: true` (sandbox on by default)
  - `filesystem.allowWrite` and `filesystem.denyWrite` defaults
  - `network.allowedDomains` base array (empty — populated by stack merging)

## Requires

Nothing. The sandbox template is a leaf node with no dependencies.

## Boundary Rules

- Must contain exactly one file: `sandbox/default.json`.
- Must conform to the SandboxTemplate schema.
- `enabled` must default to `true` (per interview decision).
- `network.allowedDomains` must start empty — domains are contributed by stack modules during merging.
- Must not contain stack-specific configuration. Stack sandbox overrides belong in `stack-data`.

## Internal Design Notes

```json
{
  "enabled": true,
  "filesystem": {
    "allowWrite": ["."],
    "denyWrite": ["/.git"]
  },
  "network": {
    "allowedDomains": []
  }
}
```

This template is the starting point. During init, the skill:
1. Reads this template
2. Merges `sandbox.filesystem` from each selected stack (union allowWrite, union denyWrite)
3. Merges `sandbox.network.allowedDomains` from each selected stack (union)
4. Writes the merged result to the `sandbox` key in settings.local.json
