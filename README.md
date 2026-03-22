# Guardrail

A Claude Code plugin that automates permission allowlists and sandbox configuration for your projects. Guardrail detects your tech stacks, generates safe default rules, and applies OS-level sandboxing — so you spend less time clicking through permission prompts and more time coding.

Without guardrail, Claude Code projects accumulate one-off permission rules from individual sessions: commit hashes, temp-file paths, and overly specific commands that clutter your settings. Guardrail replaces that drift with stack-aware, generalized allowlists and an audit tool that identifies cruft, risks, and consolidation opportunities.

## Quick Start

1. **Install**: Add guardrail to your Claude Code plugins.
2. **Navigate** to your project directory.
3. **Run** `/guardrail init` — guardrail detects your stacks, presents a menu, and writes a tuned configuration.

## Skills

### `/guardrail init`

Detects your project's tech stacks by scanning for marker files (e.g., `package.json`, `Cargo.toml`, `go.mod`), then generates a merged set of permission allow/deny rules and sandbox configuration. The flow:

1. **Discovery** — Scans for stack marker files and loads rule modules from the plugin and any project-level overrides in `.claude/guardrail/`.
2. **Interactive menu** — Presents detected stacks (pre-checked) and available stacks/rules for you to toggle:

```
Detected stacks:
  [x] python (pyproject.toml found)
  [x] node (package.json found)

Available stacks:
  [ ] rust
  [ ] go
  [ ] shell

Rule modules:
  [x] git (recommended)
  [x] web (recommended)
  [ ] docker
  (auto-included: mcp-ideate, mcp-cyberbrain)

Sandbox: [x] enabled

Adjust any selections, or proceed with these?
```

3. **Merge** — Combines the base stack, selected stacks, and selected rules into deduplicated, sorted permission arrays and sandbox config.
4. **Write** — Shows the final JSON, asks for confirmation, and writes to `.claude/settings.local.json`. If the file already exists, you can choose to merge (union new rules with existing) or replace (overwrite permissions and sandbox, preserving all other keys). A `.bak` backup is created on replace.

Pass `--no-sandbox` to disable sandbox configuration.

### `/guardrail audit`

Analyzes your existing permission rules and produces a categorized report. The audit reads both global (`~/.claude/settings.json`) and local (`.claude/settings.local.json`) settings, then classifies every local rule into one of five categories:

- **Risky** — Rules matching dangerous patterns (`rm -rf`, `sudo`, `curl | sh`, writes to `.env`/`.ssh`).
- **Redundant** — Exact duplicates or strict subsets of broader rules already present.
- **One-off / Session Artifacts** — Rules containing commit hashes, temp paths, or session-specific fragments.
- **Generalizable** — Groups of 3+ rules sharing a command prefix that can be consolidated (e.g., multiple `npm` commands to `Bash(npm:*)`).
- **Clean** — Well-scoped, unique, non-dangerous rules requiring no action.

Example report output:

```
## Audit Report: .claude/settings.local.json

**Total rules analyzed:** 24 (local) + 8 (global, read-only)

### Risky (1 rules)
  Bash(rm -rf build/:*) — allows recursive deletion

### Redundant (3 rules)
  Bash(git status:*) — subset of Bash(git:*)
  ...

### One-off / Session Artifacts (5 rules)
  Bash(git checkout abc123de:*) — contains commit hash
  ...

### Generalizable (6 rules)
  Bash(npm run build:*) → consolidate to Bash(npm:*)
  Bash(npm test:*)       → consolidate to Bash(npm:*)
  ...

### Clean (9 rules)
  Bash(cargo:*) — appropriately scoped
  ...
```

After the report, you choose a fix mode:

1. **Fix all** — Apply all recommendations automatically (risky rules still require individual confirmation).
2. **Fix one-by-one** — Step through each recommendation for per-rule approval.
3. **Export only** — Save the report to `.claude/guardrail/audit-report.md` without modifying settings.

The audit never modifies global settings — all writes go to `.claude/settings.local.json` only.

## Included Stacks

All projects automatically receive the `_base` stack. Additional stacks are activated when their detection files are found.

| Stack | Detection Files | Tools Allowed | Sandbox Domains |
|-------|----------------|---------------|-----------------|
| `_base` | *(always included)* | `ls`, `cat`, `head`, `tail`, `wc`, `find`, `grep`, `echo`, `which`, `pwd`, `date`, `mkdir`, `cp`, `mv`, `touch`, `chmod`, `diff`, `sort`, `uniq`, `xargs`, `tr`, `cut`, `tee`, `sed`, `awk`, `basename`, `dirname`, `realpath`, `env`, `export`, `printenv`, `Read` | Filesystem: write `.`, deny write `/.git` |
| `python` | `requirements.txt`, `pyproject.toml`, `setup.py`, `setup.cfg`, `Pipfile`, `*.py`, `poetry.lock`, `uv.lock` | `python3`, `python`, `pip3`, `pip`, `poetry`, `pdm`, `uv`, `pipx`, `pytest`, `tox`, `nox`, `mypy`, `ruff`, `black`, `isort`, `flake8`, `pylint`, `setuptools`, `build`, `twine`, `venv`, `virtualenv` | `pypi.org`, `files.pythonhosted.org` |
| `node` | `package.json`, `tsconfig.json`, `*.js`, `*.ts`, `*.jsx`, `*.tsx`, `yarn.lock`, `pnpm-lock.yaml` | `node`, `npm`, `npx`, `yarn`, `pnpm`, `jest`, `vitest`, `eslint`, `prettier`, `tsc`, `tsx`, `next`, `vite`, `webpack`, `esbuild`, `turbo` | `registry.npmjs.org`, `registry.yarnpkg.com` |
| `rust` | `Cargo.toml`, `Cargo.lock`, `*.rs` | `cargo`, `rustc`, `rustup`, `clippy`, `rustfmt`, `cargo-watch`, `cargo-nextest` | `crates.io`, `static.crates.io` |
| `go` | `go.mod`, `go.sum`, `*.go` | `go`, `gofmt`, `goimports`, `golint`, `golangci-lint`, `staticcheck` | `proxy.golang.org`, `sum.golang.org` |
| `shell` | `Makefile`, `*.sh`, `justfile`, `Taskfile.yml` | `make`, `just`, `bash`, `sh`, `shellcheck`, `shfmt` | *(none)* |

The `_base` stack also includes deny rules blocking: `rm -rf /`, `rm -rf ~`, `sudo`, `curl|sh`, `wget|sh`, and edits to `.env`, credentials, and `.ssh` files.

## Included Rules

| Rule | Description | Auto-included? |
|------|-------------|---------------|
| `git` | Git version control operations. Allows `git:*` but denies `git push --force`, `git push -f`, `git reset --hard`, `git clean -f`, and `git checkout -- .` | No |
| `web` | Web search and documentation fetching. Allows `WebSearch` and `WebFetch` for `docs.anthropic.com`, `github.com`, `stackoverflow.com`, and `developer.mozilla.org` | No |
| `docker` | Docker and container operations. Allows `docker`, `docker-compose`, and `docker compose` but denies `docker run --privileged` | No |
| `mcp-ideate` | Ideate plugin MCP tool permissions. Allows all `mcp__plugin_ideate_ideate-artifact-server__*` tools | Yes |
| `mcp-cyberbrain` | Cyberbrain plugin MCP tool permissions. Allows all `mcp__plugin_cyberbrain_cyberbrain__*` tools | Yes |

Auto-included rules are always applied and cannot be toggled off during init.

## Customization

You can override any built-in stack or rule by placing files in your project's `.claude/guardrail/` directory:

- **Custom stacks**: `.claude/guardrail/stacks/<name>.json`
- **Custom rules**: `.claude/guardrail/rules/<name>.json`

### Override semantics

- **Same-name replacement**: A file with the same name as a built-in file replaces the built-in entirely. For example, `.claude/guardrail/stacks/python.json` completely replaces the plugin's `python.json` — there is no deep-merge.
- **New filenames are additive**: A file with a new name (e.g., `.claude/guardrail/stacks/elixir.json`) is added to the available stacks alongside the built-ins.

### Stack JSON schema

```json
{
  "name": "string",
  "description": "string",
  "detect": { "files": ["glob patterns"], "match": "any | all" },
  "permissions": { "allow": ["..."], "deny": ["..."] },
  "sandbox": {
    "filesystem": { "allowWrite": ["..."], "denyWrite": ["..."] },
    "network": { "allowedDomains": ["..."] }
  }
}
```

### Rule JSON schema

```json
{
  "name": "string",
  "description": "string",
  "auto": false,
  "permissions": { "allow": ["..."], "deny": ["..."] }
}
```

Set `"auto": true` to make a rule always included (non-toggleable).

## How It Works

1. **Override discovery**: Guardrail loads definitions from the plugin root (`stacks/`, `rules/`, `sandbox/`), then checks for project-level overrides in `.claude/guardrail/`. Same-name project files replace plugin files entirely.
2. **Stack detection**: Each stack's `detect.files` patterns are globbed against the project root. If the `match` mode is satisfied (`"any"` = at least one pattern matches, `"all"` = every pattern must match), the stack is marked as detected.
3. **Merge algorithm**: Permissions are merged in order: `_base` first, then selected stacks alphabetically, then selected rules (including auto rules). Allow and deny arrays are unioned, deduplicated, and sorted. Sandbox arrays follow the same union/dedup/sort process.
4. **Write location**: Guardrail detects the git root via `git rev-parse --git-dir` and writes to `<git-root>/.claude/settings.local.json`. Outside a git repo, it asks you where to write.
5. **Existing settings**: Only the `permissions` and `sandbox` keys are modified. All other keys in the settings file (e.g., `env`, `hooks`, `plugins`, `mcpServers`) are preserved exactly as they are.

## Key Knowledge

### Sandbox auto-allow mode

Guardrail enables OS-level sandboxing by default. On macOS this uses Seatbelt; on Linux it uses bubblewrap. When sandboxing is active, commands that operate within the sandbox boundaries run without prompting — the sandbox itself is the security enforcement layer, so per-command prompts are unnecessary.

### Deny always wins

Claude Code enforces a strict rule: **deny rules always take precedence over allow rules**. If a command matches both an allow pattern and a deny pattern, it is denied. This is Claude Code's built-in behavior and guardrail's security foundation. You cannot accidentally override a deny rule by adding a broader allow rule.

### Global vs. local settings interaction

Claude Code combines global settings (`~/.claude/settings.json`) with local project settings (`.claude/settings.local.json`). Both sets of permissions are active simultaneously. Guardrail's audit skill reads both files to detect redundancies and conflicts, but **only modifies local settings** — global settings are never touched.

### What guardrail touches

Guardrail only modifies the `permissions` and `sandbox` keys in your settings file. All other configuration — `env`, `hooks`, `plugins`, `mcpServers`, and any other keys — are preserved without modification. Guardrail never writes to global settings, never deletes non-permission config, and never writes without your explicit confirmation.

## FAQ

**Can I use this outside a git repo?**
Yes. If guardrail cannot detect a git root, it will ask you where to write the settings file, defaulting to `<cwd>/.claude/settings.local.json`.

**How do I undo changes?**
When you choose "Replace" during init, guardrail creates a backup at `settings.local.json.bak` in the same directory. Restore from that backup to revert. For merge operations, you can manually remove the added rules from `.claude/settings.local.json`.

**Will this break my existing settings?**
No. Guardrail only touches the `permissions` and `sandbox` keys. All other keys in your settings file are preserved exactly as they are.

**What if sandboxing isn't available on my platform?**
Guardrail will display a warning. When sandboxing is unavailable, the permission rules themselves serve as the enforcement layer. The tighter permission allowlists compensate for the lack of OS-level sandbox enforcement. You can also pass `--no-sandbox` to init to skip sandbox configuration entirely.
