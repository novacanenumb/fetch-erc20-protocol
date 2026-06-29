# Fetch ERC20 Protocol Foundry Test Plan and DoD Coverage Matrix

## Objective

Define the Foundry test suite for Fetch ERC20 Protocol production so section 14 and the sandbox Definition of Done are covered by deterministic, auditable tests with explicit fixture setup and pass or fail criteria.

## Scope and assumptions

- v0 scope only: mock USDC, mock spot assets, mock oracle pricing, and a mock spot adapter.
- No yield, LP, lending, or perps adapters in the active execution path.
- Use exact integer assertions for NAV, PPS, fee splits, and share math.
- Prefer exact custom-error assertions where the contract exposes named errors.
- The spec's suggested deposit, redeem, fee, and rebalance suite names are the baseline; this plan adds missing factory, pause, oracle, security, and invariant coverage.

## Recommended test tree

```text
test/
├─ ThesisVault.factory.t.sol
├─ ThesisVault.deposit.t.sol
├─ ThesisVault.redeem.t.sol
├─ ThesisVault.fees.t.sol
├─ ThesisVault.accounting.t.sol
├─ ThesisVault.oracle.t.sol
├─ ThesisVault.pause.t.sol
├─ ThesisVault.rebalance.t.sol
├─ ThesisVault.security.t.sol
└─ ThesisVault.invariants.t.sol
```

## Fixture catalog

| Fixture | Contents | Used by |
| --- | --- | --- |
| `BaseVaultFixture` | Approved quote asset, approved spot assets, approved adapter, fresh oracle prices, valid fee split, weights that sum to 10,000, cap-compliant asset weights, unpaused state, cooldown satisfied. | Positive factory, deposit, redeem, accounting, and security tests. |
| `BootstrapDepositFixture` | Base fixture plus funded depositor, quote token approval, and an empty vault ready for the first deposit. | First-deposit, bootstrap fee, and initial PPS tests. |
| `PostBootstrapVaultFixture` | Base fixture plus one successful deposit, non-zero total supply, seeded spot positions, and known current PPS. | Second-deposit, redeem, fee, NAV, PPS, and allocation tests. |
| `BadFactoryConfigFixture` | Parameterized invalid registration inputs, such as duplicate symbols, unapproved assets, unapproved adapters, invalid fee splits, bad weight sums, and weight cap violations. | Factory negative tests. |
| `StaleOracleFixture` | Base fixture plus at least one stale price timestamp or missing freshness update. | Oracle freshness rejection tests. |
| `PausedDepositsFixture` | Base fixture with `PAUSED_DEPOSITS` enabled. | Deposit pause tests and redeem-during-deposit-pause test. |
| `PausedRebalanceFixture` | Base fixture with `PAUSED_REBALANCE` enabled. | Rebalance-only pause edge tests. |
| `PausedDepositsAndRebalanceFixture` | Base fixture with `PAUSED_DEPOSITS_AND_REBALANCE` enabled. | Combined pause edge tests. |
| `PausedAllFixture` | Base fixture with `PAUSED_ALL` enabled. | Full-pause redeem and emergency-pause tests. |
| `FeePrecisionFixture` | Base fixture plus odd-lot amounts, non-even fee splits, and values that force rounding paths. | Fee split and holder-retained PPS uplift tests. |
| `CooldownFixture` | Base fixture plus a recent rebalance timestamp inside the cooldown window. | Rebalance cooldown tests. |
| `DecimalNormalizationFixture` | Base fixture with 6-decimal quote and 18-decimal spot assets, plus non-trivial prices and balances. | Decimal normalization and accounting tests. |
| `RogueAssetFixture` | Base fixture plus a token that is not in the approved quote or spot registry. | Unapproved-asset runtime rejection tests. |
| `RogueAdapterFixture` | Base fixture plus a non-approved adapter contract. | Unapproved-adapter rejection tests. |

## Required test matrix

### Factory and registry validation

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testFactoryCreatesValidVault` | `BaseVaultFixture` | Factory accepts a fully valid config and deploys a usable vault. | Vault deployment succeeds; stored quote asset, spot assets, adapter allowlists, fee config, weights, and caps match the fixture; the vault is immediately eligible for deposit. | Any revert, mis-stored config, or missing registry linkage. |
| `testFactoryRejectsDuplicateSymbol` | `BadFactoryConfigFixture` with an already-registered symbol | Factory refuses to create a second vault with the same symbol. | Call reverts and the existing vault registration is unchanged. | A second vault is registered or the duplicate symbol is accepted. |
| `testFactoryRejectsUnapprovedQuoteAsset` | `BadFactoryConfigFixture` with an unapproved quote token | Factory blocks a quote asset that is not approved. | Call reverts before deployment. | Vault is created with an unapproved quote asset. |
| `testFactoryRejectsUnapprovedSpotAsset` | `BadFactoryConfigFixture` with an unapproved spot token | Factory blocks a spot asset that is not approved. | Call reverts before deployment. | Vault is created with an unapproved spot asset. |
| `testFactoryRejectsUnapprovedAdapter` | `BadFactoryConfigFixture` with a non-approved adapter | Factory blocks an adapter that is not approved for v0. | Call reverts before deployment. | Vault is created with a rogue adapter attached. |
| `testFactoryRejectsBadFeeSplit` | `BadFactoryConfigFixture` with an invalid creator, protocol, or holder split | Factory rejects fee splits that do not satisfy the configured bounds. | Call reverts for any split that is out of range, does not sum correctly, or exceeds the allowed fee caps. | Invalid fee math is accepted into vault state. |
| `testFactoryRejectsWeightsNotSummingTo10000` | `BadFactoryConfigFixture` with total target weights not equal to 10,000 | Factory rejects invalid target weight totals. | Call reverts before deployment. | Vault is created with a weight sum other than 10,000. |
| `testFactoryRejectsAssetWeightAboveMax` | `BadFactoryConfigFixture` with one asset above the per-asset cap | Factory rejects a configuration where one asset exceeds its cap. | Call reverts before deployment. | Vault is created with a weight above the maximum. |

### Deposit lifecycle

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testFirstDepositMintsSharesAtInitialSharePrice` | `BootstrapDepositFixture` | The first deposit mints shares at the initial share price and uses the bootstrap fee waiver path. | Shares minted equal the expected bootstrap amount; the depositor receives the shares; PPS remains at the configured initial share price after the first mint. | A bootstrap fee is charged, shares are minted from the wrong price, or the initial PPS is incorrect. |
| `testSecondDepositMintsSharesBasedOnNavPerShare` | `PostBootstrapVaultFixture` | A later deposit mints shares from current NAV per share, not the initial share price. | Shares minted equal the expected current-PPS output after fee treatment; total supply and NAV move consistently with the deposit. | The deposit uses the wrong PPS, over-mints, or under-mints shares. |
| `testDepositRevertsAfterDeadline` | `BaseVaultFixture` with an expired deadline | Deposit respects the deadline guard. | Call reverts and vault balances do not change. | Deposit succeeds after the deadline or mutates vault state. |
| `testDepositRevertsIfMinSharesOutNotMet` | `BaseVaultFixture` with `minSharesOut` above the computed output | Deposit respects slippage protection. | Call reverts before share minting; no user shares or vault positions change. | Deposit succeeds below the minimum or mints shares anyway. |
| `testDepositRevertsWhenDepositsPaused` | `PausedDepositsFixture` | Deposit is blocked when deposits are paused. | Call reverts and the vault state remains unchanged. | Deposit succeeds while deposit pause is active. |
| `testDepositRevertsWhenOracleStale` | `StaleOracleFixture` | Deposit rejects stale or missing oracle data. | Call reverts before minting shares or allocating assets. | Deposit succeeds with stale pricing data. |
| `testDepositAllocatesAcrossWeightedSpotPositions` | `PostBootstrapVaultFixture` | Deposit routes quote into spot positions according to target weights. | Post-deposit spot deltas match the configured weight ratios within integer rounding; only approved assets and approved adapter paths are used. | Allocation ignores the target weights, routes through a disallowed asset or adapter, or leaves incorrect balances. |

### Redeem lifecycle

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testRedeemBurnsShares` | `PostBootstrapVaultFixture` | Redeeming shares burns the exact input share amount. | Share balance falls by the redeemed amount and total supply decreases accordingly. | Shares are not burned or the burn amount is wrong. |
| `testRedeemReturnsQuoteAsset` | `PostBootstrapVaultFixture` | Redeeming shares returns mock USDC, not spot assets. | User quote balance increases by the expected amount, adjusted for redemption fee treatment, and the vault releases the expected quote liquidity. | Redemption pays the wrong asset or the wrong amount. |
| `testRedeemRevertsAfterDeadline` | `BaseVaultFixture` with an expired deadline | Redemption respects the deadline guard. | Call reverts and share balances do not change. | Redeem succeeds after deadline or mutates state. |
| `testRedeemRevertsIfMinQuoteOutNotMet` | `PostBootstrapVaultFixture` with `minQuoteOut` above the computed output | Redemption respects slippage protection. | Call reverts before share burn or quote transfer. | Redemption succeeds below the minimum or returns less than the floor. |
| `testRedeemWorksDuringDepositPause` | `PausedDepositsFixture` | Deposit pause must not block redeem. | Redeem succeeds, burns shares, and pays quote while deposits remain paused. | Redeem is blocked by deposit pause. |
| `testRedeemIsBlockedDuringFullPause` | `PausedAllFixture` | Full pause blocks redemption. | Call reverts and the vault state remains unchanged. | Redeem succeeds during full pause. |

### Fee accounting

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testDepositFeeSplitWorks` | `FeePrecisionFixture` | Deposit fee splits into holder, creator, and protocol portions correctly. | Holder, creator, and protocol balances match the expected split exactly; the three pieces sum to the total fee; holder-retained value remains inside the vault. | Any fee recipient receives the wrong amount, or the split does not sum exactly. |
| `testRedemptionFeeSplitWorks` | `FeePrecisionFixture` | Redemption fee splits into holder, creator, and protocol portions correctly. | Holder, creator, and protocol balances match the expected redemption split exactly; the split sums exactly to the total fee. | Any fee recipient receives the wrong amount, or the split leaks value. |
| `testHolderRetainedDepositFeeIncreasesPPS` | `FeePrecisionFixture` | The retained portion of a deposit fee stays in the vault and lifts PPS for existing holders. | PPS after the fee settles is greater than PPS before the deposit; the depositor does not get extra shares for the retained portion. | PPS does not rise, or the depositor captures the retained fee by mistake. |
| `testHolderRetainedRedemptionFeeIncreasesPPS` | `FeePrecisionFixture` | The retained portion of a redemption fee stays in the vault and lifts PPS for remaining holders. | PPS after the redemption fee settles is greater than PPS before the redemption; the retained fee is not minted to the redeemer. | PPS does not rise, or the fee is paid out in a way that bypasses holders. |
| `testCreatorFeeRecipientReceivesExpectedAmount` | `FeePrecisionFixture` | Creator fee recipient receives exactly the configured creator share. | Creator balance increase equals the calculated creator portion for both deposit and redemption fee paths that are exercised. | Creator is underpaid, overpaid, or paid from the wrong fee source. |
| `testProtocolFeeRecipientReceivesExpectedAmount` | `FeePrecisionFixture` | Protocol fee recipient receives exactly the configured protocol share. | Protocol balance increase equals the calculated protocol portion for both deposit and redemption fee paths that are exercised. | Protocol is underpaid, overpaid, or paid from the wrong fee source. |

### Accounting and normalization

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testNavEqualsIdleQuotePlusSpotPositionValues` | `PostBootstrapVaultFixture` | NAV is the sum of idle quote plus the value of all approved spot positions. | NAV matches the exact integer valuation derived from idle quote and spot holdings only. | NAV includes hidden value sources, excludes a position, or misprices holdings. |
| `testPpsEqualsNavDividedByTotalSupply` | `PostBootstrapVaultFixture` | PPS equals NAV divided by total supply when supply is non-zero. | For any non-zero supply state, PPS matches the integer division rule used by the contract, and the initial share price is used only for the zero-supply bootstrap path. | PPS is computed from the wrong denominator or drifts from the documented formula. |
| `testTotalSupplyCannotIncreaseWithoutNavIncrease` | `InvariantFixture` or a state-sequence harness | Supply growth must be backed by new capital or retained NAV, not by accounting drift. | Across the sequence, any increase in total supply is matched by a corresponding NAV increase from the same action or fee retention, and no phantom shares appear. | Supply rises from rounding error, fee bookkeeping, or any source that does not add NAV. |
| `testDecimalNormalizationWorksFor6DecimalQuoteAnd18DecimalAssets` | `DecimalNormalizationFixture` | Quote and spot decimals are normalized correctly. | Deposit, allocation, NAV, PPS, and redemption calculations remain exact after scaling between 6-decimal quote and 18-decimal spot assets. | Any off-by-one unit, scale mismatch, or precision loss appears in the accounting path. |

### Security, pause, and rebalance controls

| Test name | Fixture | Expected behavior | Pass conditions | Fail conditions |
| --- | --- | --- | --- | --- |
| `testUnapprovedAssetCannotBeUsed` | `RogueAssetFixture` | A token that is not in the approved asset registry cannot be used in the protocol flow. | Deposit, allocation, or valuation attempts that reference the rogue asset revert before any asset movement or accounting change. | The rogue asset is accepted, transferred, or valued. |
| `testUnapprovedAdapterCannotBeCalled` | `RogueAdapterFixture` | A non-approved adapter cannot be used by vault logic. | Rebalance or adapter-entry attempts revert and no position change occurs. | The rogue adapter executes or changes the vault state. |
| `testCreatorCannotDirectlyWithdrawAssets` | `BaseVaultFixture` with creator role access | The creator cannot directly withdraw vault assets outside the approved protocol flow. | Any withdrawal or recovery surface is role-gated or absent for the creator, and no quote or spot assets leave the vault on a direct creator call. | The creator can pull assets out of the vault without redeeming. |
| `testRebalanceCooldownIsEnforced` | `CooldownFixture` | Rebalance is blocked until the cooldown window has elapsed. | A rebalance attempt before the cooldown expires reverts; a later attempt after expiry succeeds. | Cooldown is ignored or the boundary behaves incorrectly. |
| `testEmergencyPauseBlocksDepositsAndRebalances` | `PausedAllFixture` | The emergency or full pause path blocks deposits and rebalances. | Deposit and rebalance calls both revert, and the vault state remains unchanged. | Either flow succeeds during the emergency pause. |

## DoD traceability

- `mock USDC deposit` -> `testFirstDepositMintsSharesAtInitialSharePrice`, `testSecondDepositMintsSharesBasedOnNavPerShare`, `testRedeemReturnsQuoteAsset`
- `weighted spot allocation` -> `testDepositAllocatesAcrossWeightedSpotPositions`
- `thesis share minting` -> `testFirstDepositMintsSharesAtInitialSharePrice`, `testSecondDepositMintsSharesBasedOnNavPerShare`
- `NAV reporting` -> `testNavEqualsIdleQuotePlusSpotPositionValues`
- `price-per-share reporting` -> `testPpsEqualsNavDividedByTotalSupply`, `testHolderRetainedDepositFeeIncreasesPPS`, `testHolderRetainedRedemptionFeeIncreasesPPS`
- `redemption to mock USDC` -> `testRedeemReturnsQuoteAsset`
- `fee split accounting` -> `testDepositFeeSplitWorks`, `testRedemptionFeeSplitWorks`, `testCreatorFeeRecipientReceivesExpectedAmount`, `testProtocolFeeRecipientReceivesExpectedAmount`
- `holder-retained fee PPS uplift` -> `testHolderRetainedDepositFeeIncreasesPPS`, `testHolderRetainedRedemptionFeeIncreasesPPS`
- `registry validation` -> all factory validation tests
- `oracle staleness rejection` -> `testDepositRevertsWhenOracleStale`, plus the edge-case oracle tests below
- `pause controls` -> the deposit, redeem, and pause tests above
- `rebalance cooldown` -> `testRebalanceCooldownIsEnforced`, plus the cooldown boundary edge case below

## Missing edge-case tests to add

These are not spelled out in section 14, but they close the obvious gaps in the spec and DoD coverage.

| Test name | Why it is missing from section 14 | Fixture | Expected behavior |
| --- | --- | --- | --- |
| `testPauseRebalanceOnlyAllowsDepositAndRedeem` | Section 14 covers deposit pause and full pause, but not the rebalance-only mode. | `PausedRebalanceFixture` | Deposits and redeems succeed, but rebalance reverts while the rebalance pause is active. |
| `testPauseDepositsAndRebalanceAllowsRedeem` | Section 14 does not enumerate the combined deposit-and-rebalance pause mode. | `PausedDepositsAndRebalanceFixture` | Deposits and rebalances revert, while redeem remains available. |
| `testRebalanceRevertsWhenOracleStale` | Section 14 names stale-oracle rejection for deposit, but rebalance also depends on fresh oracle data. | `StaleOracleFixture` | Rebalance reverts before any position change when a required oracle is stale. |
| `testOracleFreshnessBoundaryAtMaxAge` | The spec says stale prices are forbidden, but not the exact boundary behavior. | `BaseVaultFixture` with oracle age set to the freshness limit | The exact freshness boundary passes or fails consistently with the contract rule, and the result is documented. |
| `testOracleZeroPriceReverts` | The security rules forbid zero price usage, but section 14 does not test it directly. | `BaseVaultFixture` with a zero-price oracle response | Any deposit, redeem, or rebalance path that consumes the zero price reverts. |
| `testMissingOracleConfigReverts` | A missing oracle mapping is a different failure mode from a stale oracle. | `BaseVaultFixture` with one asset missing oracle config | The affected flow reverts before valuation or minting. |
| `testBootstrapDepositFeeWaived` | The deposit flow explicitly waives the fee on the first deposit, but section 14 does not isolate that rule. | `BootstrapDepositFixture` | The first deposit charges no deposit fee and still mints the expected bootstrap shares. |
| `testFeeRoundingDustIsConservedExactly` | Fee math precision and integer rounding are not spelled out in section 14. | `FeePrecisionFixture` | Gross amount equals net amount plus all fee pieces, with no lost or created dust outside the documented rounding rule. |
| `testDeadlineBoundaryAtExactTimestamp` | Section 14 only says "after deadline", not what happens exactly at the deadline. | `BaseVaultFixture` with `deadline == block.timestamp` | The exact deadline behavior is asserted once and then reused consistently across deposit, redeem, and rebalance paths. |
| `testRebalanceCooldownBoundaryAtExactTimestamp` | Section 14 requires cooldown enforcement, but not the boundary timestamp. | `CooldownFixture` at the exact cooldown boundary | The exact boundary either passes or fails in a documented way, and early calls still revert. |
| `testZeroAmountDepositAndRedeemRevert` | Zero-amount inputs are a common safety edge case and are not enumerated in section 14. | `BaseVaultFixture` | Zero-value deposit or redeem calls revert without side effects. |

## Additional spec-level gaps from Hilbert validation audit

These are explicit obligations in Jobs 7 and 8 that were not yet named as standalone test cases in this plan. They are mandatory unless the implementation packet owner documents a narrower equivalent test that proves the same behavior.

| Test name | Missing obligation | Source obligation | Expected behavior |
| --- | --- | --- | --- |
| `testRedeemRevertsWhenTotalSupplyIsZero` | Redemption must check supply state before calculating gross claim or burning shares. | Job 7.2.1 | Redeem reverts cleanly when total supply is zero, with no quote transfer, spot exit, or share mutation. |
| `testRebalanceRevertsForNonCreator` | Rebalance must enforce creator authority. | Job 8.2.1 | A non-creator caller cannot rebalance, sell positions, update target weights, or mutate cooldown state. |
| `testRebalanceRejectsArbitraryExternalCalls` | Vault logic must prevent arbitrary external calls. | Job 8.3.3 | Rebalance cannot execute a non-approved external target or payload, and vault balances remain unchanged. |
| `testFactoryRejectsYieldLpPerpAdapterKinds` | v0 factory deployment must reject non-spot adapter kinds. | Jobs 1.4 and 8.3.4 | Factory rejects adapters marked `YIELD`, `LP`, or `PERP` before vault deployment or registry attachment. |
| `testRebalanceRejectsYieldLpPerpAdapterKinds` | v0 rebalance must reject non-spot adapter kinds. | Jobs 1.4, 8.2.3, and 8.3.4 | Rebalance rejects `YIELD`, `LP`, or `PERP` adapter routes before asset movement. |

## Current validation limit

Forge validation cannot currently be proven from this production checkout. The production root has no `foundry.toml`, no Solidity sources, and no `.t.sol` tests. The spec's DoD command points at `sandbox/root/app/thesis-protocol`, which is not present in this production root. Until that package exists and is lease-approved, this document is a coverage plan, not passing test evidence.

## Acceptance rule

The Foundry test plan is complete when every required row above has a deterministic test case in the recommended suite tree and every edge-case row is either implemented or explicitly deferred with a written reason.
