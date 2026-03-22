# Policies: Sandbox

## P-1: Sandbox enabled by default
All generated configurations enable sandboxing with auto-allow mode. This is the primary mechanism for reducing permission prompts while maintaining OS-level enforcement.
- **Derived from**: GP-1 (Maximize Autonomy Within Safety Boundaries)
- **Established**: planning phase
- **Status**: active

## P-2: Warn on unsupported platforms, compensate with tighter rules
If sandboxing is unavailable (unsupported platform), the skill warns the user and should note that permission rules become the primary safety mechanism.
- **Derived from**: GP-5 (Defense in Depth)
- **Established**: planning phase
- **Status**: active

## P-3: Network domains are additive from stacks
The sandbox template starts with empty `allowedDomains`. Each selected stack contributes its package registry domains. No stack can remove domains added by another stack.
- **Derived from**: GP-6 (Composable and Extensible)
- **Established**: planning phase
- **Status**: active
