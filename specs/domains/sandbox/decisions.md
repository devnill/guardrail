# Decisions: Sandbox

## D-1: Auto-allow mode enabled by default
- **Decision**: Generated sandbox configs use auto-allow mode where sandboxed Bash commands auto-approve without prompts
- **Rationale**: This is the single biggest lever for reducing permission prompts. OS-level enforcement provides the safety net.
- **Source**: steering/interviews/plan/_full.md — sandbox auto-allow question
- **Status**: settled

## D-2: Default filesystem boundaries
- **Decision**: allowWrite: ["."] (project directory), denyWrite: ["/.git"] (git database)
- **Rationale**: Agents need to write files in the project but should not manipulate git internals directly
- **Source**: plan/architecture.md — sandbox template design
- **Status**: settled
