# Fetch ERC20 Protocol Release Environment

## Purpose

Migrated ERC20 protocol application placeholders from 06_WORKSPACE/00_APPLICATIONS.

## Current Status

This isolated organisation/application environment root was created during the `06_WORKSPACE` deprecation migration.

## Environment Role

`release` - Release candidate state for packaging and migration notes.

## Setup

1. Start from the repository root.
2. Read `/AGENT_INIT.md`, `/PROJECT_MAP.md`, `/MANIFEST.json`, and `orpheus.toml`.
3. Verify the active task lease and path claim before modifying this root.

## Safety Boundaries

- This root does not grant access to Orpheus core engine internals.
- This root does not grant access to sibling organisation or application roots.
- Local `.agents/` state is private runtime context and must not be committed.
- Secrets must stay in sanctioned environment or secure-store flows.

## Navigation

- `AGENT_INIT.md`
- `index.md`
- `docs/`
- `app/`, `src/`, `utils/`
