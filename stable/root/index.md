# index.md - Agent Navigation Index

## Project Identity

Organisation: `ORG_ORPHEUS`
Application: `fetch-erc20-protocol`
Environment: `stable`

## Purpose

Migrated ERC20 protocol application placeholders from 06_WORKSPACE/00_APPLICATIONS.

## Current Entry Points

- `app/`
- `src/`
- `utils/`
- `docs/`
- `README.md`
- `AGENT_INIT.md`

## Environment Role

`stable`

## Allowed Actions

Read, document, validate, and modify files inside this environment root only when a task lease grants the path.

## Restricted Actions

- Do not access Orpheus core engine files without explicit bootstrap and path-lease authority.
- Do not access another organisation or app root without a permission contract.
- Do not commit `.agents/` contents.
- Do not store secrets, private keys, or raw provider tokens here.

## External Dependencies

No external tooling is implicitly approved. Tooling must be granted through the Orpheus root bootstrap and permission model.

## Handoff Notes

Continue by reading `README.md`, `AGENT_INIT.md`, `docs/HANDOFF.md`, and upstream Orpheus root bootstrap files.
