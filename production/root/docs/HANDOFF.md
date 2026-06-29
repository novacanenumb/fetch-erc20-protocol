# Fetch ERC20 Protocol Production Handoff

```yaml
handoff:
  objective: "Maintain fetch-erc20-protocol in production under ORG_ORPHEUS."
  current_status: "Production root now contains the external build prompt, collapsible organisation task tree append, static embedded-browser orchestration demo, and admin-dashboard nested-org projection for Fetch ERC20 Protocol."
  files_touched:
    - "production/root/fetch-erc20-spec.md"
    - "production/root/index.md"
    - "production/root/docs/HANDOFF.md"
    - "production/root/app/fetch-erc20-demo/index.html"
    - "production/root/app/fetch-erc20-demo/README.md"
    - "05_APPLICATIONS/01_OPERATOR/03_DASHBOARDS/nested_org_control_plane/apps/web/src/features/dashboard-pages/DashboardRouter.tsx"
    - "05_APPLICATIONS/01_OPERATOR/03_DASHBOARDS/nested_org_control_plane/packages/protocol-types/mock_protocol_snapshot.json"
  decisions_made:
    - "This root is isolated and must bootstrap through the Orpheus root protocol."
    - "production/root is the current planning, review, and orchestration surface."
    - "Protocol implementation remains sandbox-root gated unless nova or Fetch grants a separate written implementation or promotion lease."
    - "Three GPT-5.4-mini agent lanes are active: Tesla for compliance, Raman for architecture decomposition, and Hilbert for QA/test planning."
    - "Tesla's compliance caveats were integrated into the task tree authority notes."
    - "Raman's dependency chain and ten-packet implementation cut were integrated into the task tree."
    - "Hilbert's feature-gate test matrix and boundary cases were integrated into the Foundry validation job."
    - "Six live GPT-5.4-mini workers are assigned: Tesla medium, Raman high, Hilbert xhigh, Noether medium, Turing high, and Curie xhigh."
    - "Current worker assignment ids are Tesla=019f137c-fb39-7443-a60c-2229581b5f61, Raman=019f137c-fce3-7741-bf7f-091a63e57ea7, Hilbert=019f137c-fe37-78f3-9ca7-752fba4b9b9e, Noether=019f137c-ffb7-7563-9cb6-41dc026e14f3, Turing=019f137d-0137-7862-b35a-d6f9061630f7, Curie=019f137d-0291-77c1-9896-24c4d6013d51."
    - "Three Claude API lanes are represented as secure-gated provider agents without secret exposure or live credential assumptions."
    - "Claude readiness check found no ANTHROPIC_API_KEY, no CLAUDE_API_KEY, and no ORPHEUS_CLAUDE_MANAGED_AGENTS_ENABLED=true; Claude API lanes remain secure-gated."
    - "Local demo server was started at http://127.0.0.1:8787/ for live viewing."
    - "Meitner's protocol trace was integrated as the demo's full v0 trace preset."
    - "Euclid's Claude API safety guidance was integrated by keeping Claude lane UI non-secret-bearing and secure-gated."
    - "Fetch ERC20 Protocol is projected in the external main admin dashboard as a child organisation under ORG_ORPHEUS with its own scoped task tree."
    - "The production root does not own the admin dashboard artifact; dashboard evidence is recorded as an external control-plane projection."
  tests_run: []
  validation:
    - "Invoke-WebRequest http://127.0.0.1:8787/ returned HTTP 200 and confirmed expected title, GPT agents, Claude lanes, state inspector, and search controls."
    - "node --check passed for the embedded JavaScript extracted from app/fetch-erc20-demo/index.html."
    - "Claude readiness was checked by presence only; no secret values were printed."
    - "External admin dashboard dev server at http://127.0.0.1:8791/ returned HTTP 200."
    - "External rendered Edge/Playwright validation confirmed Fetch ERC20 Nested Organisation, Fetch ERC20 Task Tree, Fetch ERC20 Protocol, ORG_ORPHEUS, Packet jobs, and Blocked metrics are visible."
  known_issues: []
  unresolved_risks:
    - "Environment-specific runtime has not been validated unless a later handoff says otherwise."
    - "The spec contains a deliberate authority split: current orchestration is in production/root, but implementation is still sandbox/root by default."
    - "No Claude API connector or approved Claude credential is exposed in this Codex environment, so the three Claude API agents remain secure-gated rather than live external workers."
    - "Claude secure-gating is currently a static/UI-level representation, not a live provider runtime enforcement path."
    - "Admin dashboard strict build currently fails on broader pre-existing TypeScript/module-resolution issues around @orpheus/protocol-types; runtime Vite display is verified."
  next_agent: "main orchestrator or delegated agent with verified path lease"
  next_required_action: "Collect the six GPT worker handoffs, then either provide an approved Claude API execution path or keep the Claude lanes secure-gated while starting Packet 1 only after confirming the intended implementation write root."
```
