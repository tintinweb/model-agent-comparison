# Project Scoping Report

## Executive Summary
- **Project Type**: Solidity DeFi Vault Aggregation System
- **Total Source Files (core)**: 9
- **Primary Language**: Solidity 0.8.23 (Upgradeable pattern)
- **Complexity Assessment**: Medium (multi-layer inheritance + strategy composition + upgradeability + fee & timelock logic)
- **Estimated Audit Effort**: 6 - 8 auditor days (single pass) or 2 auditors * 1 week including remediation review

## Scope
Core in-scope contracts (excluding tests, mocks, scripts):
- `src/BaseVault.sol` (abstract upgradeable ERC4626 base w/ auth, pause, caps)
- `src/PerformanceVault.sol` (abstract performance fee extension)
- `src/SizeMetaVault.sol` (meta vault orchestrating strategies + timelock + ordering & rebalancing)
- `src/IBaseVault.sol` (interface)
- `src/utils/Auth.sol` (separate upgradeable access control root)
- `src/utils/Timelock.sol` (abstract timelock mixin)
- `src/strategies/AaveStrategyVault.sol` (strategy interacting with Aave pool & aToken)
- `src/strategies/ERC4626StrategyVault.sol` (wrapper strategy around external ERC4626)
- `src/strategies/CashStrategyVault.sol` (minimal strategy holding idle assets)

### Source Metrics (Selected)
- Normalized SLOC: 684
- Public Functions: 47 / External: 20
- State Variables: 18 (15 public) — surface for invariant checks
- Deployables: SizeMetaVault, Auth, CashStrategyVault, AaveStrategyVault, ERC4626StrategyVault

### Deployable Contracts Purpose
- `Auth`: Central role & upgrade authorization contract
- `SizeMetaVault`: Entry point for users; allocates deposits across strategies; manages performance fees & timelock-governed admin ops
- `CashStrategyVault`: Passive holding strategy, likely for buffer/liquidity
- `AaveStrategyVault`: Yield strategy depositing into Aave (risk: external protocol interactions)
- `ERC4626StrategyVault`: Generic wrapper integrating any ERC4626 underlying (composability & integration risk)

## Technical Overview
### Architecture
Layered abstraction:
1. Base Layer: `BaseVault` (ERC4626Upgradeable + UUPSUpgradeable + access & pausability) centralizing deposit/mint logic caps and reentrancy guards.
2. Performance Layer: `PerformanceVault` extends BaseVault with performance fee accounting via high-water mark and fee recipient tracking.
3. Meta Layer: `SizeMetaVault` extends PerformanceVault + Timelock for multi-strategy management (add/remove/reorder, rebalance, skim) and administrative delay controls.
4. Strategies: Each strategy vault itself is a vault (inherits BaseVault) providing a consistent interface (ERC4626) enabling the meta vault to treat them polymorphically.
5. Access Control: Externalized in `Auth` to allow multiple vaults to share governance & roles.
6. Timelock: Abstract mixin providing per-action timelocking; integrated only in meta vault.

### Key Components
- Upgradeability via UUPS (risk: _authorizeUpgrade correctness & role separation)
- Timelocked admin parameter changes (risk: bypass, duplicate function selectors)
- Strategy array management (ordering affects allocation logic)
- Performance fee minting & high-water mark accuracy
- Asset cap logic & dead assets accounting
- External protocol integration (Aave & arbitrary ERC4626)

### External Dependencies
- OpenZeppelin Upgradeable suite (access, ERC4626, reentrancy, pausing, multicall)
- Aave V3 interfaces (IPool, IAToken)
- External arbitrary ERC4626 targets (integration variability)

## Security Scope
### Attack Surface / Entry Points
- User deposit/withdraw flows in meta vault & strategies
- Rebalance / skim operations (may move large asset amounts)
- Strategy addition/removal/reordering (affects funds routing)
- Performance fee adjustments & fee minting (economic extraction risk)
- Timelock duration changes
- Upgrade functions in Auth and vaults

### Trust Relationships
- Auth contract roles (Admin, Pauser?, Upgrader) — central authority
- Meta vault holds user funds; strategies are controlled by meta vault logic
- External protocols introduce trust in Aave and external ERC4626 implementations

### Typical Actors
- Governance / Admin (auth role): can configure fees, strategies, caps, upgrades
- Keeper / Operator: may trigger rebalance / skim (if not restricted) — verify roles
- Users (depositors): supply and redeem assets
- External Protocols: Aave pool, external ERC4626 vaults

### User & Data Flows
1. Deposit Flow: User -> SizeMetaVault deposit -> allocation across strategies -> strategy internal deposit -> accounting update & potential performance fee logic
2. Withdrawal Flow: User redeem -> meta vault determines strategy withdrawals (ordering & proportional logic) -> underlying retrieval -> transfer to user
3. Rebalance Flow: Authorized caller triggers `rebalance` -> withdraw from some strategies, deposit into others based on target ordering/weights (need to inspect logic specifics in implementation review phase)

### Critical Security Areas
- Accuracy & safety of strategy add/remove/reorder (array manipulation bounds, duplicates)
- Reentrancy protections on external call boundaries (deposit/withdraw in strategies & meta vault)
- Fee calculation integrity (prevent front-running or fee inflation exploiting high-water mark resets)
- Timelock completeness (all sensitive functions covered? any function omitted?)
- Upgrade authorization isolation & initialization guarding
- External protocol return value / failure handling (Aave, ERC4626 underlying)
- Asset cap enforcement & dead asset accounting correctness

## Audit Recommendations
### Methodology
- Static analysis (slither, semgrep custom rules, upgradeability patterns)
- Differential testing with property-based fuzz (echidna) for invariants: totalAssets consistency, fee monotonicity, timelock delays
- Fork tests for Aave integration behaviors

### Priority Focus Areas
1. Strategy management (state integrity & invariants)
2. Fee minting & distribution correctness
3. Timelock coverage & race scenarios
4. Upgradeability & initializer misuse
5. External call failure & reentrancy edges across nested ERC4626 interactions

### Tooling
- Slither detectors (upgradeability, reentrancy, shadowed vars)
- Echidna properties (existing properties dir suggests some present)
- Foundry fuzz tests expansion for edge allocations

## Complexity Assessment
### Factors
- Moderate inheritance depth (Base -> Performance -> Meta + Timelock mixin)
- Multi-contract upgrade surface (Auth + multiple vaults)
- External protocol variability (arbitrary ERC4626)
- Economic logic (performance fees, high-water mark)

### Risk Areas
- Economic mis-accounting leading to fee misallocation
- Strategy removal during pending timelock or during active rebalance
- Asset accounting drift (deadAssets vs totalAssets vs strategies sums)

## Effort Estimation
- nSLOC 684 -> Base factor ~0.5 auditor-days per 100 nSLOC baseline = ~3.4 days
- Complexity multiplier ~1.7 (upgradeable + multi-strategy + economic layer) => ~5.8 days
- Add integration & fuzz design overhead: +1 - 2 days
- Total: 6 - 8 auditor-days (initial assessment) excluding remediation

### Resource Requirements
- 1 lead auditor + 1 support (optional) for parallel economic vs technical track
- Specialist: economic modeling (optional for fee fairness)

## Suggested Next Steps Before Full Audit
- Provide threat model for fee & strategy manipulation
- Enumerate required privileged functions & confirm timelock coverage list
- Supply deployment / upgrade procedures documentation
- Confirm role separation in Auth (list role IDs & intended holders)

## Conclusion
Codebase is a focused medium-size upgradeable DeFi vault aggregation framework with layered abstractions and moderate complexity concentrated in meta vault composition & fee accounting. Primary audit emphasis should be on economic correctness, privileged operation safeguards (timelock & roles), and robustness of external protocol interactions.
