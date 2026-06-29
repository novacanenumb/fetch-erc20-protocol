# Fetch ERC20 Protocol Directive

## Organisation Overview

`fetch-erc20-protocol` is an isolated application root under `ORG_ORPHEUS`. Migrated ERC20 protocol application placeholders from 06_WORKSPACE/00_APPLICATIONS.

## Mission Statement

Deliver scoped application work without bypassing Orpheus bootstrap authority, path leases, review gates, or organisation isolation.

## Scope

This directive applies to every environment root under `06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/`.

## Rules

- Bootstrap through root `/AGENT_INIT.md` before acting.
- Verify organisation, application, environment, and path lease.
- Keep private `.agents/` state untracked.
- Treat production as high-integrity and sandbox as experimental.
- Fail closed when permissions, ownership, or external-tool authority are unclear.

## Public Records

Public organisation records live in this application directory and its environment-local `docs/` folders.

## Private Records

Private runtime state belongs in ignored `.agents/` directories or sanctioned secure-store flows only.
