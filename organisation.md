# Fetch ERC20 Protocol Organisation Structure

## Purpose

Describe the full operating structure for `fetch-erc20-protocol` inside `ORG_ORPHEUS`.

## Operating Model

```text
Orpheus Orchestrator
|- Board of Observers
|- CEO
|  `- COO
|     |- Heads of Department / VCs
|     |  `- Workers
|     `- Environment Operators
|        |- stable/root
|        |- release/root
|        |- production/root
|        |- staging/root
|        |- sandbox/root
|        `- scripts/root
```

## Operators

Operator assignments are not created by this migration. Future assignments must name authority, scope, permissions, handoff rules, and review requirements.

## Role Boundaries

| Role | Authority | Output |
| --- | --- | --- |
| Board of Observers | Oversight and policy review | acceptance or rejection evidence |
| CEO | Strategic alignment and final acceptance | approved objectives |
| COO | Sequencing, delivery tracking, task assignment | scoped tasks and handoffs |
| Heads of Department / VCs | Functional ownership | plans, reviews, and resourcing decisions |
| Workers | Implementation and validation | scoped changes and test evidence |
