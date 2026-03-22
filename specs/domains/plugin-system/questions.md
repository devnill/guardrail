# Questions: Plugin System

## Q-1: Should guardrail detect installed plugins automatically?
- **Question**: Currently mcp-ideate and mcp-cyberbrain rules are `auto: true` regardless of whether those plugins are installed. Should guardrail check which plugins are actually enabled and only auto-include relevant MCP rules?
- **Source**: architecture design — rule module auto behavior
- **Impact**: Including rules for uninstalled plugins is harmless but adds noise to the generated config
- **Status**: open
- **Reexamination trigger**: User feedback about unnecessary MCP rules in generated config
