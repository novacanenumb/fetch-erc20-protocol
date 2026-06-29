# Fetch ERC20 Protocol Embedded Demo

This static browser app visualizes the Fetch ERC20 Protocol orchestration task tree, six GPT-5.4-mini subagent lanes, three secure-gated Claude API agent lanes, implementation packets, and a local-only thesis-vault accounting trace.

## Scope

- Runs as a local static browser app.
- Does not request or store secrets.
- Does not make live provider calls.
- Does not deploy contracts or write protocol implementation files.
- Represents Claude API agents as secure-gated lanes until approved credential references exist.
- Current Claude readiness check found no `ANTHROPIC_API_KEY`, no `CLAUDE_API_KEY`, and no `ORPHEUS_CLAUDE_MANAGED_AGENTS_ENABLED=true`, so the demo correctly keeps Claude lanes secure-gated.
- Claude lane UI uses non-secret governance labels only: `approved secure-store reference required` and `no approved reference loaded`.
- The protocol trace includes the full local-only v0 sequence from INIT through bootstrap deposit, mark-to-market, fee-bearing deposit, redeem, pause modes, and stale-oracle rejection.

## Entry Point

```text
index.html
```

## Validation

Open `index.html` in the embedded browser and confirm:

- The overview metrics render.
- The assigned agent lanes render.
- The organisation task tree expands and collapses.
- Task tree search and status filtering update visible jobs.
- Packet selection updates the packet detail panel.
- Protocol trace buttons update NAV, shares, PPS, holder fees, pause mode, and logs.
- `Load full v0 trace` loads the complete local-only trace and updates the visible vault state.
- The state inspector mirrors selected packet, filters, lease state, vault state, and Claude API lane status.
- Claude API lanes show secure-gated status without secret values.
