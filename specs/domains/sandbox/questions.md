# Questions: Sandbox

## Q-1: Should sandbox config be auditable?
- **Question**: V1 audit only analyzes permission rules. Should it also analyze sandbox configuration (e.g., overly broad allowWrite, missing denyWrite for sensitive dirs)?
- **Source**: architecture design — scope boundary
- **Impact**: Sandbox misconfigurations could undermine the security model
- **Status**: open
- **Reexamination trigger**: After v1 ships and adversarial testing begins

## Q-2: Unix socket restrictions
- **Question**: Should the default sandbox template restrict unix socket access? Research indicates unix sockets can enable sandbox escape.
- **Source**: steering/research/claude-code-permissions.md — sandbox limitations
- **Impact**: Potential sandbox bypass if unix sockets are unrestricted
- **Status**: open
- **Reexamination trigger**: During security audit phase
