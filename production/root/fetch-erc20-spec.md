# External Agent Build Prompt — Fetch ERC20 Protocol

You are operating inside the Orpheus project repository.

Repository root:

```text
N:\Development\Orpheus\orpheus-production
```

You are building the isolated organisation application:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/
```

The application is owned and governed by the following primary stakeholders:

```text
Fetch — CEO — 75% equity
nova — The Orchestrator — 25% equity
```

Treat this stakeholder model as project-governance metadata for the build environment. Do not alter stakeholder ownership, authority, or equity records unless explicitly instructed by the user.

---

# 1. Mandatory Containment Rule

You must perform all implementation work inside:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/
```

You may read root-level policy files for context, but you must not write outside the application root.

You must not create, move, modify, or delete files in:

```text
00_AI_CORE/
01_MCP_CORE/
02_CRYPTO_CORE/
03_ENGINE/
04_GOVERNANCE/
05_APPLICATIONS/
06_WORKSPACE/
07_DOCUMENTATION/
```

You must not create new project roots under:

```text
06_WORKSPACE/
```

`06_WORKSPACE` is deprecated and retained only for migration compatibility.

If a required file appears to belong outside the allowed application root, stop and create a written proposal instead of editing it.

---

# 2. Required Boot Sequence

Before making changes, read these files if present:

```text
AGENT_INIT.md
06_ORGANISATIONS/README.md
06_ORGANISATIONS/PERMISSION_MODEL.md
06_ORGANISATIONS/ENVIRONMENT_MODEL.md
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/stakeholders.md
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/organisation.md
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/directive.md
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/policies.md
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/orchestration.md
```

If any application-level files are missing, create them inside:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/
```

Do not create them elsewhere.

---

# 3. Application Objective

Build a local-first ERC20 thesis-token protocol.

The protocol lets creators deploy ERC20 thesis tokens backed by smart-contract vaults that hold weighted baskets of approved spot assets.

v0 must prove one core primitive:

```text
A thesis token’s share price accurately reflects the quote-denominated value of its underlying approved spot positions.
```

v0 must only support:

```text
spot thesis vaults
mock USDC quote asset
mock spot assets
mock oracle pricing
mock spot adapter
Foundry tests
local-only development
```

v0 must not support:

```text
yield adapters
LP adapters
lending adapters
perps adapters
leverage
real DEX routing
real oracle integration
mainnet deployment
governance deployment
frontend marketplace
cross-chain behaviour
production keys
```

Advanced strategies may be documented as future adapter layers, but they must not be implemented in the v0 code path.

---

# 4. Environment Layout

The application must follow the Orpheus nested environment model:

```text
06_ORGANISATIONS/
`- ORG_ORPHEUS/
   `- fetch-erc20-protocol/
      |- stakeholders.md
      |- organisation.md
      |- directive.md
      |- policies.md
      |- orchestration.md
      |- stable/root/
      |- release/root/
      |- production/root/
      |- staging/root/
      |- sandbox/root/
      `- scripts/root/
```

Use the environments as follows:

```text
sandbox/root/
  Active development only.

staging/root/
  Tested integration candidate only.

release/root/
  Release candidate only.

production/root/
  Approved production snapshot only.

stable/root/
  Stable specs, canonical architecture notes, and frozen reference docs.

scripts/root/
  Organisation-level validation scripts only.
```

For the first milestone, write only to:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/scripts/root/
```

Write to `scripts/root/` only if creating validation scripts for the application boundary.

---

# 5. Sandbox Build Structure

Inside:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/
```

create or maintain:

```text
.agents/
app/
utils/
src/
docs/
README.md
AGENT_INIT.md
index.md
```

Place the Foundry protocol implementation under:

```text
sandbox/root/app/thesis-protocol/
```

Expected protocol package structure:

```text
sandbox/root/app/thesis-protocol/
|- foundry.toml
|- README.md
|- src/
|  |- ThesisFactory.sol
|  |- ThesisVault.sol
|  |- registry/
|  |  |- AssetRegistry.sol
|  |  `- AdapterRegistry.sol
|  |- oracle/
|  |  |- IOracleRouter.sol
|  |  `- MockOracleRouter.sol
|  |- adapters/
|  |  |- IThesisAdapter.sol
|  |  `- MockSpotAdapter.sol
|  |- mocks/
|  |  `- MockERC20.sol
|  `- libraries/
|     |- BasisPoints.sol
|     |- Errors.sol
|     `- Types.sol
`- test/
   |- ThesisFactory.t.sol
   |- ThesisVault.deposit.t.sol
   |- ThesisVault.redeem.t.sol
   |- ThesisVault.accounting.t.sol
   |- ThesisVault.security.t.sol
   |- ThesisVault.fees.t.sol
   `- ThesisVault.rebalance.t.sol
```

---

# 6. Core Protocol Specification

## 6.1 ThesisFactory

The factory deploys new thesis vaults.

It must:

```text
validate quote asset approval
validate spot asset approval
validate adapter approval
validate oracle availability
validate target weights sum to 10,000
validate target weights do not exceed asset-level caps
validate fee split
validate symbol uniqueness
deploy ThesisVault
emit VaultCreated
```

It must not:

```text
custody user funds
rebalance vaults
bypass registry checks
override vault accounting
deploy non-spot v0 vaults
```

---

## 6.2 ThesisVault

The vault is also the ERC20 share token.

It must:

```text
accept quote asset deposits
mint thesis shares
redeem thesis shares
track spot positions
calculate NAV
calculate price per share
apply mint/redeem fees
split fees between holders, creator, and protocol
retain holder fees inside the vault
enforce pause state
enforce deadline checks
enforce minSharesOut
enforce minQuoteOut
enforce approved adapters
enforce oracle freshness
enforce rebalance cooldown
```

Required external functions:

```solidity
function deposit(
    uint256 quoteAmount,
    address receiver,
    uint256 minSharesOut,
    uint256 deadline
) external returns (uint256 sharesOut);

function redeem(
    uint256 shares,
    address receiver,
    uint256 minQuoteOut,
    uint256 deadline
) external returns (uint256 quoteOut);

function totalNav() public view returns (uint256 navQuote);

function pricePerShare() public view returns (uint256 pps);

function rebalance(
    PositionUpdate[] calldata updates,
    uint256 deadline
) external;

function pause(uint8 mode) external;

function unpause() external;
```

---

## 6.3 AssetRegistry

Stores approved asset policies.

Required policy shape:

```solidity
struct AssetPolicy {
    bool allowedAsQuote;
    bool allowedAsSpot;
    uint16 maxVaultWeightBps;
    uint256 minLiquidityQuote;
    bytes32 oracleId;
    uint32 maxOracleStaleness;
    bool active;
}
```

v0 must reject:

```text
unapproved quote assets
unapproved spot assets
inactive assets
assets with missing or stale oracle configuration
assets exceeding maxVaultWeightBps
```

---

## 6.4 AdapterRegistry

Stores approved adapters.

Required adapter kind enum:

```solidity
enum AdapterKind {
    SPOT,
    YIELD,
    LP,
    PERP
}
```

v0 must only permit:

```text
AdapterKind.SPOT
```

Yield, LP, and PERP may exist as enum values for future compatibility, but they must not be usable in v0 vault deployments or v0 rebalances.

---

## 6.5 MockOracleRouter

Use a local mock oracle only.

It must:

```text
allow test-controlled price updates
return quote-denominated prices
return price decimals
return last updated timestamp
reject stale prices
reject zero prices
support 6-decimal quote asset and 18-decimal spot assets
```

Do not integrate Chainlink, Pyth, DEX TWAPs, or any production oracle in v0.

---

## 6.6 MockSpotAdapter

Use a local mock spot adapter only.

It must:

```text
simulate quote-to-asset conversion
simulate asset-to-quote conversion
use mock oracle prices
allow controlled slippage simulation in tests
report position value in quote terms
avoid arbitrary external calls
only handle approved spot assets
```

No real DEX routing in v0.

---

# 7. Fee and Incentive Model

v0 supports deposit and redemption fees only.

Do not implement:

```text
management fees
performance fees
benchmark rewards
creator rankings
```

Required fee config:

```solidity
struct FeeConfig {
    uint16 depositFeeBps;
    uint16 redemptionFeeBps;

    uint16 holderFeeShareBps;
    uint16 creatorFeeShareBps;
    uint16 protocolFeeShareBps;

    address creatorFeeRecipient;
    address protocolFeeRecipient;
}
```

Validation:

```text
depositFeeBps <= 100
redemptionFeeBps <= 100
holderFeeShareBps + creatorFeeShareBps + protocolFeeShareBps == 10,000
creatorFeeRecipient != address(0)
protocolFeeRecipient != address(0)
```

Holder incentive invariant:

```text
Holder-retained fees must increase NAV/share for existing or remaining holders and must not mint extra shares to the user who generated the fee.
```

---

# 8. Accounting Rules

Use quote-denominated NAV.

For v0:

```text
NAV = idleQuote + sum(spot position values)
```

Price per share:

```text
if totalSupply == 0:
    PPS = initialSharePrice
else:
    PPS = NAV / totalSupply
```

Use consistent precision. Prefer `1e18` internal precision for price-per-share math while correctly normalising 6-decimal quote tokens and 18-decimal spot tokens.

---

# 9. Deposit Flow

Required logic:

```text
1. Check deadline.
2. Check deposits are not paused.
3. Calculate navBefore.
4. Transfer quoteAmount from user.
5. If totalSupply == 0, waive deposit fee for bootstrap simplicity.
6. Otherwise calculate depositFee.
7. Split depositFee into holderFee, creatorFee, protocolFee.
8. Retain holderFee inside vault.
9. Transfer or accrue creatorFee and protocolFee.
10. Allocate net investable amount into spot positions.
11. Calculate navAfter.
12. addedNavForMint = navAfter - navBefore - holderFee.
13. Mint shares against addedNavForMint.
14. Require sharesOut >= minSharesOut.
15. Emit Deposit and FeesApplied.
```

---

# 10. Redemption Flow

Required logic:

```text
1. Check deadline.
2. Check vault is not fully paused.
3. Calculate navBefore.
4. Calculate grossClaimQuote = shares * navBefore / totalSupply.
5. Calculate redemptionFee.
6. Split redemptionFee into holderFee, creatorFee, protocolFee.
7. Exit enough positions into quote.
8. Burn shares.
9. Retain holderFee in vault.
10. Transfer creatorFee to creator recipient.
11. Transfer protocolFee to protocol recipient.
12. Transfer quoteOut = grossClaimQuote - redemptionFee to receiver.
13. Require quoteOut >= minQuoteOut.
14. Emit Redeem and FeesApplied.
```

Redemptions should remain available during deposit pause and rebalance pause where possible.

---

# 11. Pause Modes

Use simple pause modes:

```text
0 = ACTIVE
1 = PAUSED_DEPOSITS
2 = PAUSED_REBALANCE
3 = PAUSED_DEPOSITS_AND_REBALANCE
4 = PAUSED_ALL
```

Rules:

```text
PAUSED_DEPOSITS blocks deposits only.
PAUSED_REBALANCE blocks rebalances only.
PAUSED_DEPOSITS_AND_REBALANCE blocks deposits and rebalances.
PAUSED_ALL blocks deposits, rebalances, and redemptions.
```

---

# 12. Rebalancing v0

Rebalancing should be simple and constrained.

A creator may rebalance only if:

```text
vault is not paused for rebalancing
rebalance cooldown has passed
deadline has not expired
all assets are approved
all adapters are approved spot adapters
all oracles are fresh
new weights sum to 10,000
new weights are within asset-level caps
```

Acceptable local v0 implementation:

```text
sell current mock spot positions back to quote
allocate quote according to new target weights
```

This is inefficient but acceptable for local testing.

---

# 13. Security Rules

Use:

```text
OpenZeppelin SafeERC20
OpenZeppelin ReentrancyGuard
OpenZeppelin ERC20
custom errors
strict config validation
```

Must prevent:

```text
creator direct withdrawal of vault assets
creator arbitrary external calls
unapproved asset usage
unapproved adapter usage
stale oracle usage
zero price usage
minting shares against holder-retained fees
yield usage in v0
LP usage in v0
perps usage in v0
writes outside the application environment
```

---

# 14. Required Tests

Write Foundry tests for:

```text
Factory creates valid vault
Factory rejects duplicate symbol
Factory rejects unapproved quote asset
Factory rejects unapproved spot asset
Factory rejects unapproved adapter
Factory rejects bad fee split
Factory rejects weights not summing to 10,000
Factory rejects asset weight above max

First deposit mints shares at initial share price
Second deposit mints shares based on NAV/share
Deposit reverts after deadline
Deposit reverts if minSharesOut not met
Deposit reverts when deposits paused
Deposit reverts when oracle stale
Deposit allocates across weighted spot positions

Redeem burns shares
Redeem returns quote asset
Redeem reverts after deadline
Redeem reverts if minQuoteOut not met
Redeem works during deposit pause
Redeem is blocked during full pause

Deposit fee split works
Redemption fee split works
Holder-retained deposit fee increases PPS
Holder-retained redemption fee increases PPS
Creator fee recipient receives expected amount
Protocol fee recipient receives expected amount

NAV equals idle quote plus spot position values
PPS equals NAV divided by total supply
Total supply cannot increase without NAV increase
Decimal normalization works for 6-decimal quote and 18-decimal assets

Unapproved asset cannot be used
Unapproved adapter cannot be called
Creator cannot directly withdraw assets
Rebalance cooldown is enforced
Emergency pause blocks deposits and rebalances
```

---

# 15. Definition of Done

The sandbox milestone is complete when:

```bash
cd 06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/app/thesis-protocol
forge test
```

passes and the implementation proves:

```text
mock USDC deposit
weighted spot allocation
thesis share minting
NAV reporting
price-per-share reporting
redemption to mock USDC
fee split accounting
holder-retained fee PPS uplift
registry validation
oracle staleness rejection
pause controls
rebalance cooldown
```

Do not promote to staging, release, production, or stable without explicit approval from nova or Fetch.

---

# 16. Agent Failure Behaviour

If you encounter ambiguity, unsafe scope, or missing authority:

```text
stop
do not guess
do not write outside scope
create a local note under sandbox/root/.agents/
explain the blocked action
propose the minimum safe next step
```

Do not silently expand scope.

Do not implement advanced strategy adapters.

Do not touch `06_WORKSPACE`.


# Fetch ERC20 Protocol — Organisation Containment Spec

## 1. Application Identity

Application name:

```text
fetch-erc20-protocol
```

Organisation:

```text
ORG_ORPHEUS
```

Application root:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/
```

Application class:

```text
isolated external-agent build environment
```

Protocol type:

```text
local-first ERC20 thesis-token vault protocol
```

Primary stakeholders:

```text
Fetch — CEO — 75% equity
nova — The Orchestrator — 25% equity
```

---

## 2. Stakeholder Authority

## Fetch

Role:

```text
CEO
```

Equity:

```text
75%
```

Authority:

```text
strategic direction
product approval
promotion approval
protocol-level governance approval
production-readiness approval
```

## nova

Role:

```text
The Orchestrator
```

Equity:

```text
25%
```

Authority:

```text
build orchestration
agent coordination
implementation sequencing
sandbox execution approval
spec clarification
technical review
```

## External Agents

Role:

```text
delegated implementation workers
```

Equity:

```text
0%
```

Authority:

```text
execute scoped build tasks inside approved environment roots only
```

External agents may not:

```text
change stakeholder records
change equity records
promote builds across environments
write outside the application root
create new workspace roots
introduce production integrations
alter organisation-level governance
```

---

## 3. Environment Containment

All agent implementation work must occur inside:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/
```

The allowed write surface for v0 is:

```text
fetch-erc20-protocol/stakeholders.md
fetch-erc20-protocol/organisation.md
fetch-erc20-protocol/directive.md
fetch-erc20-protocol/policies.md
fetch-erc20-protocol/orchestration.md
fetch-erc20-protocol/sandbox/root/
fetch-erc20-protocol/scripts/root/
```

The main protocol build must live in:

```text
fetch-erc20-protocol/sandbox/root/app/thesis-protocol/
```

No implementation files should be placed directly in the repository root.

No implementation files should be placed in `06_WORKSPACE`.

---

## 4. Environment Promotion Model

## sandbox/root

Purpose:

```text
active development, tests, mocks, local-only experiments
```

Allowed:

```text
code scaffolding
contract implementation
Foundry tests
mock assets
mock oracle
mock adapter
local docs
agent notes
```

Not allowed:

```text
mainnet deployment
production keys
release claims
external protocol integrations
```

## staging/root

Purpose:

```text
integration candidate after sandbox tests pass
```

Promotion requires:

```text
explicit approval from nova or Fetch
passing test report
scope review
security checklist
```

## release/root

Purpose:

```text
release candidate snapshot
```

Promotion requires:

```text
explicit approval from Fetch or delegated governance
staging validation
documented changelog
```

## production/root

Purpose:

```text
approved production snapshot only
```

Promotion requires:

```text
explicit stakeholder approval
security review
deployment policy review
```

## stable/root

Purpose:

```text
stable specs and frozen reference docs
```

Allowed:

```text
canonical architecture notes
approved specs
audited references
```

---

## 5. Application File Responsibilities

## stakeholders.md

Must contain:

```text
stakeholder names
roles
equity split
authority boundaries
external agent limitations
```

## organisation.md

Must contain:

```text
application identity
relationship to ORG_ORPHEUS
environment model
ownership model
promotion model
```

## directive.md

Must contain:

```text
current objective
v0 implementation scope
non-goals
success criteria
build sequence
```

## policies.md

Must contain:

```text
containment rules
write boundaries
security rules
dependency rules
key-management rules
advanced-adapter restrictions
```

## orchestration.md

Must contain:

```text
agent roles
handoff rules
review flow
failure behaviour
promotion gates
test requirements
```

---

## 6. Protocol v0 Boundary

v0 is spot-only.

Allowed v0 modules:

```text
ThesisFactory
ThesisVault
AssetRegistry
AdapterRegistry
MockOracleRouter
MockSpotAdapter
MockERC20
BasisPoints library
Types library
Errors library
Foundry tests
```

Explicitly disallowed v0 modules:

```text
YieldAdapter
LpAdapter
LendingAdapter
PerpsAdapter
real DEX integration
real oracle integration
mainnet deployment scripts
frontend marketplace
creator ranking engine
benchmark reward engine
governance token
cross-chain bridge
```

Future adapter concepts may be documented, but not compiled into the active v0 execution path.

---

## 7. External-Agent Operating Rules

External agents must:

```text
read boot files before editing
confirm target environment path
write only inside allowed paths
prefer small testable commits
run tests after implementation changes
record unresolved issues in sandbox/root/.agents/
fail closed on unclear authority
```

External agents must not:

```text
write to deprecated 06_WORKSPACE
modify root governance files
modify unrelated applications
add hidden dependencies
use production credentials
create network deployments
install unrelated frameworks
expand beyond spot v0
```

---

## 8. Minimum Local Validation

The protocol is not considered valid until the following command passes:

```bash
cd 06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/sandbox/root/app/thesis-protocol
forge test
```

Minimum validation requirements:

```text
factory deployment works
invalid factory configs revert
deposit mints correct shares
redeem burns correct shares
NAV is correct
PPS is correct
fees split correctly
holder-retained fees increase PPS
oracle staleness reverts
unapproved assets revert
unapproved adapters revert
pause modes work
rebalance cooldown works
```

---

## 9. Canonical v0 Success Condition

The v0 protocol is successful when:

```text
A user can deposit mock USDC into a creator-launched thesis vault, receive ERC20 thesis shares, observe accurate NAV/share, and redeem shares back into mock USDC, with all accounting, fee, oracle, registry, and pause invariants tested locally.
```

---

## 10. Non-Negotiable Constraint

Do not build advanced strategy adapters before the spot vault primitive is correct.

The correct sequence is:

```text
1. organisation containment
2. local sandbox setup
3. registries
4. mock oracle
5. mock spot adapter
6. factory
7. vault accounting
8. deposits
9. redemptions
10. fees
11. pause/security controls
12. rebalancing
13. tests
14. staging proposal
```

---

## 11. Organisation Task Tree Append

This append converts the external build prompt into an executable organisation task tree. It is the coordination surface for delegated agents and must remain aligned with the containment rules above.

Current isolated orchestration root:

```text
06_ORGANISATIONS/ORG_ORPHEUS/fetch-erc20-protocol/production/root/
```

`production/root/` is the current planning, review, and orchestration surface for this task. It is not an implicit protocol-code implementation target.

Protocol implementation remains gated by explicit task lease and the v0 environment rules in this specification. The implementation fence remains `sandbox/root/` unless nova or Fetch grants a separate written exception or promotion approval. If a delegated task requires writing outside its approved root, the agent must stop and record a proposal instead of widening scope.

<details>
<summary>Fetch ERC20 Protocol Organisation Task Tree</summary>

### Job 0 - Governance and Containment Control

- [ ] Task 0.1 - Confirm active lease and root authority
  - [ ] Subtask 0.1.1 - Record current operator instruction naming `production/root/` as the isolated working environment.
  - [ ] Subtask 0.1.2 - Treat `production/root/` as planning/review authority only unless a separate implementation or promotion lease is granted.
  - [ ] Subtask 0.1.3 - Flag conflicts between production-root orchestration and sandbox-root implementation rules before code changes.
- [ ] Task 0.2 - Protect stakeholder and governance metadata
  - [ ] Subtask 0.2.1 - Keep Fetch as CEO with 75% equity.
  - [ ] Subtask 0.2.2 - Keep nova as The Orchestrator with 25% equity.
  - [ ] Subtask 0.2.3 - Prevent external agents from modifying ownership, equity, or promotion authority.
- [ ] Task 0.3 - Enforce forbidden write surfaces
  - [ ] Subtask 0.3.1 - Do not write to `00_AI_CORE/`, `01_MCP_CORE/`, `02_CRYPTO_CORE/`, `03_ENGINE/`, `04_GOVERNANCE/`, `05_APPLICATIONS/`, `06_WORKSPACE/`, or `07_DOCUMENTATION/`.
  - [ ] Subtask 0.3.2 - Do not create new project roots under `06_WORKSPACE/`.
  - [ ] Subtask 0.3.3 - Route unresolved authority issues to `sandbox/root/.agents/` or handoff documentation inside the approved environment.

Owner: Tesla, GPT-5.4-mini medium, Compliance/Audit Agent.

Acceptance: every implementation job has an approved write root, owner, blocker rule, and promotion gate before coding starts.

### Job 1 - Organisation Build Map

- [ ] Task 1.1 - Maintain environment skeleton
  - [ ] Subtask 1.1.1 - Preserve `stable/root/`, `release/root/`, `production/root/`, `staging/root/`, `sandbox/root/`, and `scripts/root/`.
  - [ ] Subtask 1.1.2 - Keep production-root artifacts as orchestration and approved reference material only unless a production implementation lease is granted.
  - [ ] Subtask 1.1.3 - Keep scripts-root reserved for validation scripts.
- [ ] Task 1.2 - Define canonical build package
  - [ ] Subtask 1.2.1 - Use `app/thesis-protocol/` as the Foundry package root.
  - [ ] Subtask 1.2.2 - Track required package files: `foundry.toml`, `README.md`, `src/`, and `test/`.
  - [ ] Subtask 1.2.3 - Keep active v0 modules limited to factory, vault, registries, mock oracle, mock spot adapter, mocks, libraries, and tests.
- [ ] Task 1.3 - Maintain local documentation
  - [ ] Subtask 1.3.1 - Document purpose, scope, non-goals, architecture, data flow, permissions, failure modes, and validation.
  - [ ] Subtask 1.3.2 - Avoid duplicate README or generic instruction sprawl.
  - [ ] Subtask 1.3.3 - Keep handoff notes specific enough for delegated continuation.

Owner: Main orchestrator.

Acceptance: the environment map, package map, and handoff map are clear enough for a worker to begin a bounded implementation task without reading unrelated repo roots.

### Job 2 - Protocol Architecture Decomposition

- [ ] Task 2.1 - Define module boundaries
  - [ ] Subtask 2.1.1 - Assign `ThesisFactory` to deployment and pre-flight config validation.
  - [ ] Subtask 2.1.2 - Assign `ThesisVault` to share token behavior, NAV, PPS, deposit, redeem, rebalance, fees, and pause controls.
  - [ ] Subtask 2.1.3 - Assign `AssetRegistry` and `AdapterRegistry` to approval policy only, not accounting.
  - [ ] Subtask 2.1.4 - Assign `MockOracleRouter` and `MockSpotAdapter` to local-only pricing and conversion simulation.
- [ ] Task 2.2 - Define shared data structures
  - [ ] Subtask 2.2.1 - Place `AssetPolicy`, `FeeConfig`, `PositionUpdate`, and asset target weight structs in shared types.
  - [ ] Subtask 2.2.2 - Place `AdapterKind` in shared types or the adapter registry interface.
  - [ ] Subtask 2.2.3 - Define custom errors for config, oracle, asset, adapter, deadline, pause, slippage, fee, and rebalance failures.
- [ ] Task 2.3 - Define accounting invariants
  - [ ] Subtask 2.3.1 - Implement quote-denominated `NAV = idleQuote + sum(spot position values)`.
  - [ ] Subtask 2.3.2 - Implement `PPS = initialSharePrice` for empty supply and `NAV / totalSupply` otherwise.
  - [ ] Subtask 2.3.3 - Normalize 6-decimal quote assets and 18-decimal spot assets with consistent internal precision.
  - [ ] Subtask 2.3.4 - Prevent minting shares against holder-retained fees.
- [ ] Task 2.4 - Freeze implementation dependency chain
  - [ ] Subtask 2.4.1 - Start with containment, package scaffold, shared libraries, mock ERC20, and local test harness.
  - [ ] Subtask 2.4.2 - Build registries before oracle, adapter, factory, or vault money movement.
  - [ ] Subtask 2.4.3 - Build mock oracle before mock spot adapter.
  - [ ] Subtask 2.4.4 - Build factory after registries, oracle, and adapter interfaces are stable.
  - [ ] Subtask 2.4.5 - Build vault accounting before deposit, redeem, fee, pause, security, or rebalance flows.
  - [ ] Subtask 2.4.6 - Run cross-cutting test hardening after feature-slice tests exist.
- [ ] Task 2.5 - Assign artifact ownership boundaries
  - [ ] Subtask 2.5.1 - Keep production-root docs as source material and orchestration artifacts.
  - [ ] Subtask 2.5.2 - Keep sandbox implementation artifacts owned by one implementation packet at a time where possible.
  - [ ] Subtask 2.5.3 - Split shared `ThesisVault.sol` work by responsibility: accounting, deposit/redeem, fees/pause/security, then rebalance.
  - [ ] Subtask 2.5.4 - Own feature tests beside the feature slice they validate, with final integration hardening last.

Owner: Raman, GPT-5.4-mini high, Architecture Breakdown Agent.

Acceptance: every contract has one primary responsibility, every shared type has one owner, and every accounting invariant has a corresponding test job.

### Job 3 - Registry and Configuration Layer

- [ ] Task 3.1 - Build asset registry policy
  - [ ] Subtask 3.1.1 - Store `allowedAsQuote`, `allowedAsSpot`, `maxVaultWeightBps`, `minLiquidityQuote`, `oracleId`, `maxOracleStaleness`, and `active`.
  - [ ] Subtask 3.1.2 - Reject inactive, unapproved, missing-oracle, stale-oracle, and overweight assets.
  - [ ] Subtask 3.1.3 - Expose read helpers for factory and vault validation.
- [ ] Task 3.2 - Build adapter registry policy
  - [ ] Subtask 3.2.1 - Store approved adapters and their `AdapterKind`.
  - [ ] Subtask 3.2.2 - Permit only `AdapterKind.SPOT` in v0 deployment and rebalance paths.
  - [ ] Subtask 3.2.3 - Keep `YIELD`, `LP`, and `PERP` as enum compatibility only.
- [ ] Task 3.3 - Build config validation gates
  - [ ] Subtask 3.3.1 - Validate target weights sum to 10,000 BPS.
  - [ ] Subtask 3.3.2 - Validate asset-level caps.
  - [ ] Subtask 3.3.3 - Validate fee split and nonzero fee recipients.

Owner: Future implementation worker lane A.

Acceptance: invalid quote asset, invalid spot asset, invalid adapter, bad weights, overweight asset, and bad fee split all fail before a usable vault is deployed.

### Job 4 - Mock Oracle and Mock Spot Adapter Layer

- [ ] Task 4.1 - Build mock oracle router
  - [ ] Subtask 4.1.1 - Allow test-controlled price updates.
  - [ ] Subtask 4.1.2 - Return quote-denominated price, decimals, and last update timestamp.
  - [ ] Subtask 4.1.3 - Reject zero and stale prices.
  - [ ] Subtask 4.1.4 - Support 6-decimal quote and 18-decimal spot assets.
- [ ] Task 4.2 - Build mock spot adapter
  - [ ] Subtask 4.2.1 - Simulate quote-to-asset conversion from oracle price.
  - [ ] Subtask 4.2.2 - Simulate asset-to-quote conversion from oracle price.
  - [ ] Subtask 4.2.3 - Support controlled slippage simulation in tests.
  - [ ] Subtask 4.2.4 - Avoid arbitrary external calls and real DEX routing.
- [ ] Task 4.3 - Connect adapter validation
  - [ ] Subtask 4.3.1 - Require approved spot assets.
  - [ ] Subtask 4.3.2 - Require approved spot adapters.
  - [ ] Subtask 4.3.3 - Require fresh oracle data before valuation or conversion.

Owner: Future implementation worker lane B.

Acceptance: all pricing and conversion behavior is deterministic, local-only, and test-controlled.

### Job 5 - Factory Deployment Layer

- [ ] Task 5.1 - Build factory deployment entrypoint
  - [ ] Subtask 5.1.1 - Validate quote asset approval.
  - [ ] Subtask 5.1.2 - Validate spot asset approval.
  - [ ] Subtask 5.1.3 - Validate adapter approval and `SPOT` kind.
  - [ ] Subtask 5.1.4 - Validate oracle availability and freshness policy.
  - [ ] Subtask 5.1.5 - Validate target weights, asset caps, fee split, and symbol uniqueness.
- [ ] Task 5.2 - Deploy vault
  - [ ] Subtask 5.2.1 - Pass immutable or constructor config into `ThesisVault`.
  - [ ] Subtask 5.2.2 - Emit `VaultCreated`.
  - [ ] Subtask 5.2.3 - Avoid user-fund custody and post-deploy accounting overrides.

Owner: Future implementation worker lane A.

Acceptance: factory creates only valid v0 spot vaults and rejects every invalid config class before deployment.

### Job 6 - Vault Accounting and Share Token Layer

- [ ] Task 6.1 - Build vault state model
  - [ ] Subtask 6.1.1 - Track quote asset, spot assets, target weights, adapters, registry references, oracle router, fee config, pause mode, and rebalance timestamps.
  - [ ] Subtask 6.1.2 - Track spot positions by asset.
  - [ ] Subtask 6.1.3 - Use ERC20 shares as the vault token.
- [ ] Task 6.2 - Build NAV and PPS views
  - [ ] Subtask 6.2.1 - Calculate idle quote balance.
  - [ ] Subtask 6.2.2 - Sum spot position values in quote terms.
  - [ ] Subtask 6.2.3 - Return normalized `totalNav()`.
  - [ ] Subtask 6.2.4 - Return normalized `pricePerShare()`.
- [ ] Task 6.3 - Preserve supply invariant
  - [ ] Subtask 6.3.1 - Mint shares only when added NAV justifies it.
  - [ ] Subtask 6.3.2 - Burn shares on redemption.
  - [ ] Subtask 6.3.3 - Prove total supply cannot increase without NAV increase.

Owner: Future implementation worker lane C.

Acceptance: NAV, PPS, and supply behavior are deterministic across bootstrap, deposit, fee, redeem, and rebalance scenarios.

### Job 7 - Deposit, Redemption, and Fee Flows

- [ ] Task 7.1 - Implement deposit flow
  - [ ] Subtask 7.1.1 - Check deadline, deposit pause, min shares, registry approval, adapter approval, and oracle freshness.
  - [ ] Subtask 7.1.2 - Transfer quote from user.
  - [ ] Subtask 7.1.3 - Waive bootstrap deposit fee when total supply is zero.
  - [ ] Subtask 7.1.4 - Split non-bootstrap deposit fee into holder, creator, and protocol portions.
  - [ ] Subtask 7.1.5 - Retain holder fees inside the vault and invest net quote into weighted spot positions.
  - [ ] Subtask 7.1.6 - Mint shares against `navAfter - navBefore - holderFee`.
- [ ] Task 7.2 - Implement redemption flow
  - [ ] Subtask 7.2.1 - Check deadline, full pause, min quote, and supply state.
  - [ ] Subtask 7.2.2 - Calculate gross claim from current NAV and total supply.
  - [ ] Subtask 7.2.3 - Split redemption fee into holder, creator, and protocol portions.
  - [ ] Subtask 7.2.4 - Exit enough spot positions to quote.
  - [ ] Subtask 7.2.5 - Burn shares and transfer net quote to receiver.
  - [ ] Subtask 7.2.6 - Keep redemptions available during deposit pause and rebalance pause.
- [ ] Task 7.3 - Validate fee incentive model
  - [ ] Subtask 7.3.1 - Enforce deposit and redemption fee caps of 100 BPS.
  - [ ] Subtask 7.3.2 - Enforce fee-share total of 10,000 BPS.
  - [ ] Subtask 7.3.3 - Prove holder-retained fees increase PPS for existing or remaining holders.

Owner: Future implementation worker lane C.

Acceptance: deposit, redeem, fee split, and holder-retained PPS uplift are proven by focused tests.

### Job 8 - Pause, Rebalance, and Security Controls

- [ ] Task 8.1 - Implement pause modes
  - [ ] Subtask 8.1.1 - Support `ACTIVE`, `PAUSED_DEPOSITS`, `PAUSED_REBALANCE`, `PAUSED_DEPOSITS_AND_REBALANCE`, and `PAUSED_ALL`.
  - [ ] Subtask 8.1.2 - Block deposits in deposit pause modes.
  - [ ] Subtask 8.1.3 - Block rebalances in rebalance pause modes.
  - [ ] Subtask 8.1.4 - Block redemptions only during full pause.
- [ ] Task 8.2 - Implement constrained rebalance
  - [ ] Subtask 8.2.1 - Enforce creator authority.
  - [ ] Subtask 8.2.2 - Enforce cooldown and deadline.
  - [ ] Subtask 8.2.3 - Validate assets, adapters, oracles, weights, and caps.
  - [ ] Subtask 8.2.4 - Sell current mock spot positions back to quote.
  - [ ] Subtask 8.2.5 - Allocate quote to new target weights.
- [ ] Task 8.3 - Implement security guardrails
  - [ ] Subtask 8.3.1 - Use SafeERC20, ReentrancyGuard, ERC20, and custom errors.
  - [ ] Subtask 8.3.2 - Prevent direct creator asset withdrawal.
  - [ ] Subtask 8.3.3 - Prevent arbitrary external calls.
  - [ ] Subtask 8.3.4 - Prevent unapproved, stale, zero-price, yield, LP, or perps usage in v0.

Owner: Future implementation worker lane B.

Acceptance: pause, rebalance, and security tests cover every allowed and disallowed state transition.

### Job 9 - Foundry Test and Validation Matrix

- [ ] Task 9.1 - Build factory tests
  - [ ] Subtask 9.1.1 - Valid vault deployment succeeds.
  - [ ] Subtask 9.1.2 - Duplicate symbol, unapproved quote, unapproved spot, unapproved adapter, bad fee split, bad weights, and overweight asset revert.
- [ ] Task 9.2 - Build deposit tests
  - [ ] Subtask 9.2.1 - First deposit mints at initial share price.
  - [ ] Subtask 9.2.2 - Second deposit mints from NAV/share.
  - [ ] Subtask 9.2.3 - Deadline, minSharesOut, deposit pause, stale oracle, and weighted allocation behavior are covered.
- [ ] Task 9.3 - Build redemption tests
  - [ ] Subtask 9.3.1 - Redeem burns shares and returns quote.
  - [ ] Subtask 9.3.2 - Deadline, minQuoteOut, deposit-pause availability, and full-pause blocking are covered.
- [ ] Task 9.4 - Build fee tests
  - [ ] Subtask 9.4.1 - Deposit and redemption fee split works.
  - [ ] Subtask 9.4.2 - Creator and protocol recipients receive expected amounts.
  - [ ] Subtask 9.4.3 - Holder-retained deposit and redemption fees increase PPS.
- [ ] Task 9.5 - Build accounting tests
  - [ ] Subtask 9.5.1 - NAV equals idle quote plus spot values.
  - [ ] Subtask 9.5.2 - PPS equals NAV divided by total supply.
  - [ ] Subtask 9.5.3 - Decimal normalization is correct for 6-decimal quote and 18-decimal assets.
  - [ ] Subtask 9.5.4 - Total supply cannot increase without NAV increase.
- [ ] Task 9.6 - Build security and rebalance tests
  - [ ] Subtask 9.6.1 - Unapproved assets and adapters cannot be used.
  - [ ] Subtask 9.6.2 - Creator cannot directly withdraw assets.
  - [ ] Subtask 9.6.3 - Rebalance cooldown is enforced.
  - [ ] Subtask 9.6.4 - Emergency pause blocks deposits and rebalances.
- [ ] Task 9.7 - Build pause and oracle boundary tests
  - [ ] Subtask 9.7.1 - `testPauseRebalanceOnlyAllowsDepositAndRedeem` proves rebalance-only pause keeps deposit and redeem behavior available.
  - [ ] Subtask 9.7.2 - `testPauseDepositsAndRebalanceAllowsRedeem` proves combined deposit/rebalance pause still allows redemptions.
  - [ ] Subtask 9.7.3 - `testRebalanceRevertsWhenOracleStale` proves rebalance cannot use stale pricing.
  - [ ] Subtask 9.7.4 - `testOracleFreshnessBoundaryAtMaxAge` proves the exact staleness cutoff.
  - [ ] Subtask 9.7.5 - `testOracleZeroPriceReverts` proves zero prices fail before value movement.
  - [ ] Subtask 9.7.6 - `testMissingOracleConfigReverts` proves missing oracle policy fails before deployment or conversion.
- [ ] Task 9.8 - Build fee precision and timestamp boundary tests
  - [ ] Subtask 9.8.1 - `testBootstrapDepositFeeWaived` proves the first deposit fee waiver is explicit.
  - [ ] Subtask 9.8.2 - `testFeeRoundingDustIsConservedExactly` proves fee rounding does not leak NAV.
  - [ ] Subtask 9.8.3 - `testDeadlineBoundaryAtExactTimestamp` proves deadline equality behavior.
  - [ ] Subtask 9.8.4 - `testRebalanceCooldownBoundaryAtExactTimestamp` proves cooldown equality behavior.
  - [ ] Subtask 9.8.5 - `testZeroAmountDepositAndRedeemRevert` proves zero-value inputs fail cleanly.
- [ ] Task 9.9 - Run local validation
  - [ ] Subtask 9.9.1 - Run `forge test` from the protocol package root.
  - [ ] Subtask 9.9.2 - Record output, failures, and unresolved risks in handoff notes.
  - [ ] Subtask 9.9.3 - Do not promote to staging, release, production, or stable without explicit approval.

Owner: Hilbert, GPT-5.4-mini xhigh, QA/Test Planning Agent.

Acceptance: each required test from the spec maps to a named test job, pause/oracle/precision edge cases have explicit coverage, and the final DoD command passes before any promotion proposal.

### Job 10 - Agent Orchestration and Handoff

- [ ] Task 10.1 - Maintain three-agent lanes
  - [ ] Subtask 10.1.1 - Tesla owns compliance, permission caveats, production-root/sandbox-root conflict tracking, and failure behavior.
  - [ ] Subtask 10.1.2 - Raman owns architecture decomposition, module ownership, invariant mapping, and implementation order.
  - [ ] Subtask 10.1.3 - Hilbert owns test-job expansion, acceptance criteria, and validation coverage.
- [ ] Task 10.2 - Convert lanes into implementation packets
  - [ ] Subtask 10.2.1 - Assign disjoint write sets before any coding.
  - [ ] Subtask 10.2.2 - Keep registries/oracle/adapter work separate from factory/vault work until interfaces are stable.
  - [ ] Subtask 10.2.3 - Keep tests mapped to the feature slice they prove.
  - [ ] Subtask 10.2.4 - Packet 1: scaffold, shared libraries, mock ERC20, Foundry configuration, and package README.
  - [ ] Subtask 10.2.5 - Packet 2: asset and adapter registries.
  - [ ] Subtask 10.2.6 - Packet 3: oracle interface and mock oracle.
  - [ ] Subtask 10.2.7 - Packet 4: thesis adapter interface and mock spot adapter.
  - [ ] Subtask 10.2.8 - Packet 5: factory deployment surface and factory tests.
  - [ ] Subtask 10.2.9 - Packet 6: vault accounting core and accounting tests.
  - [ ] Subtask 10.2.10 - Packet 7: deposit and redemption flows with matching tests.
  - [ ] Subtask 10.2.11 - Packet 8: fees, pause modes, security gates, and matching tests.
  - [ ] Subtask 10.2.12 - Packet 9: constrained rebalance and rebalance tests.
  - [ ] Subtask 10.2.13 - Packet 10: integration fixtures, regression coverage, final `forge test`, and staging proposal draft.
- [ ] Task 10.3 - Maintain handoff discipline
  - [ ] Subtask 10.3.1 - Record files touched, decisions made, tests run, known issues, unresolved risks, next agent, and next required action.
  - [ ] Subtask 10.3.2 - Stop on ambiguous authority, unsafe scope, or requested writes outside the approved root.
  - [ ] Subtask 10.3.3 - Treat green validation as evidence only for the coverage it actually proves.
- [ ] Task 10.4 - Maintain dependency map
  - [ ] Subtask 10.4.1 - Require Packet 1 before every implementation packet.
  - [ ] Subtask 10.4.2 - Require Packet 2 before oracle, adapter, factory, vault, and validation packets.
  - [ ] Subtask 10.4.3 - Require Packet 3 before adapter, factory, vault accounting, and oracle-staleness tests.
  - [ ] Subtask 10.4.4 - Require Packet 4 before factory deployment, deposits, redemptions, and rebalance.
  - [ ] Subtask 10.4.5 - Require Packet 6 before deposit, redemption, fee, pause, security, and rebalance changes.
  - [ ] Subtask 10.4.6 - Require feature-slice tests before final integration hardening.

Owner: Main orchestrator.

Acceptance: each subagent has a distinct lane, no two agents own the same write set, and every lane can be promoted into a bounded implementation packet.

</details>
