# AGENT_INIT.md - ORG_ORPHEUS / fetch-erc20-protocol / production

This is an isolated organisation application root inside the Orpheus system.

## Scope

Organisation: `ORG_ORPHEUS`
Application: `fetch-erc20-protocol`
Environment: `production`
Local root: `06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/production/root/`

## Bootstrap Rule

Agents must not initialise from this file alone. Agents must redirect to the Orpheus root bootstrap protocol and initialise through the approved external chain.

Required upstream references:

- `/AGENT_INIT.md`
- `/PROJECT_MAP.md`
- `/00_AI_CORE/`
- `/01_MCP_CORE/`
- Any active bootstrap, guardrail, permission, or governance files defined by the current Orpheus root.

## Isolation Rule

This organisation root is isolated from Orpheus core engine internals, other organisations, sibling app roots, private `.agents/` state, and unapproved external tooling.

## Fail-Closed Rule

If an agent cannot verify permissions, bootstrap chain, or current project scope, it must stop and request authorisation.
