# Codebase Scoping Plan

## Project Assessment
- **Project Type**: Solidity Smart Contract System
- **Expected Complexity**: Medium to High
- **Target Technology**: Blockchain/DeFi Vault System
- **Framework**: Foundry (evidenced by foundry.toml)

## Initial Observations
- Project name: "size-meta-vault" suggests a meta-vault architecture
- Contains strategy vaults (Aave, Cash, ERC4626)
- Has comprehensive testing setup (local, fork, property tests)
- Uses formal verification tools (Echidna, Medusa)
- Has deployment scripts and broadcast history

## Analysis Strategy
### Tools to Use:
- [x] chonky-workspace-find_files (discover all relevant files)
- [ ] chonky-solidity-metrics-analyze_solidity_files_get_overview (Solidity metrics)
- [ ] chonky-solidity-deployable-contracts (identify deployable contracts)
- [ ] chonky-solidity-contract-structure (architecture analysis)
- [ ] chonky-solidity-metrics-get_contract_call_graph (call relationships)
- [ ] chonky-solidity-metrics-get_contract_inheritance_graph (inheritance analysis)
- [ ] chonky-solidity-function-analysis (function categorization)

## Expected Deliverables
- SCOPING_REPORT.md (comprehensive scope analysis)
- Architecture overview of meta-vault system
- Complexity assessment based on metrics
- Audit effort estimation
- Key security areas identification

## Scope Boundaries
### In Scope
- Core vault contracts in /src
- Strategy implementations
- Authentication and access control
- Performance and fee mechanisms

### Out of Scope (Initially)
- Test files (/test directory)
- Deployment scripts (/script directory)
- External dependencies (/lib directory)
- Configuration files

## Analysis Plan Phases
1. **Discovery**: Find all .sol files, categorize by purpose
2. **Metrics**: Generate comprehensive Solidity metrics
3. **Architecture**: Map contract relationships and inheritance
4. **Security**: Identify attack surfaces and trust boundaries
5. **Documentation**: Create comprehensive scoping report
