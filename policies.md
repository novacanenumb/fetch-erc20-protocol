# Fetch ERC20 Protocol Policies

## Enforcement Policy

Policy violations must be recorded in handoff evidence and escalated to the relevant reviewer or governance authority. Agents must stop rather than continue with unclear authority.

## Required Policies

| Policy | Rule | Enforcement |
| --- | --- | --- |
| Bootstrap | Root bootstrap is mandatory | stop if not verified |
| Path scope | Writes require a valid lease | reject unleased changes |
| Secrets | No plaintext secrets in app roots | remove and report |
| Isolation | No sibling or core access by default | require explicit contract |
| Promotion | Environment promotion requires validation | block promotion without evidence |
| Runtime state | `.agents/` and runtime files are local-only | keep ignored and unstaged |
