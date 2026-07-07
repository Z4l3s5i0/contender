### Uber Workload Implementation Report (Updated)

### Summary
This report describes the implementation of the Uber workload scenario and campaign in Contender, based on the `minion-eurosys23` benchmark artifacts.

### Changes
- **Scenario Update (`scenarios/uber.toml`)**:
    - Renamed the contract to `ContractUber` as found in the source code.
    - Updated the interaction function to `checkDistance()`.
    - Integrated the real Solidity contract bytecode (previously using `SpamMe` placeholder).
- **Campaign Verification (`campaigns/uber_workload.toml`)**:
    - Confirmed that the TPS stages align with the `workload-uber.yaml` file (810, 840, 885, 900, 850 total TPS across all regions).
    - The existing campaign uses the aggregated TPS from the 10-region Diablo workload, which provides a more granular transition profile.

### Findings from `minion-eurosys23`
- **Contract Logic**: The `ContractUber` maintains state for 10,000 drivers (`number_of_drivers`). The `checkDistance()` function iterates through all 10,000 drivers to find the closest one to a client, involving a `sqrt` calculation and coordinate comparisons. This is a highly compute-intensive (CPU-bound) transaction.
- **Constructor**: The constructor performs 20,000+ storage writes (driver coordinates) and expensive `sha256` hashes. Deployment of this contract will likely exceed standard gas limits and take significant time.
- **Function Mapping**: The Diablo workload referred to the function as `uber:find`, but the Solidity implementation uses `checkDistance()`. The Python `arguments` script in the folder confirmed `checkDistance()` is the intended entry point.

### What is Missing
- **Gas Limit Adjustments**: Given the 10,000-driver loop, the `checkDistance()` function will consume a vast amount of gas. The runner's gas configuration may need to be increased.
- **Account Stake**: The Diablo workload specifies 10,000 accounts with specific stakes. Contender uses its own signer pool mechanism; if specific account balances are required, they must be set during the `contender setup` phase.

### Regional Capability (Diablo Parity)
- **Feature Implemented**: Contender now supports per-mix `rpc_url` overrides in campaigns.
- **Diablo Imitation**: This allows imitating Diablo's "regions" by defining multiple mixes within a stage, each targeting a different RPC endpoint while contributing to the overall TPS rate.
- **Example**:
  ```toml
  [[spam.stage.mix]]
  scenario = "scenario:uber.toml"
  share_pct = 50.0
  rpc_url = "http://region-1-node:8545"

  [[spam.stage.mix]]
  scenario = "scenario:uber.toml"
  share_pct = 50.0
  rpc_url = "http://region-2-node:8545"
  ```

### Implementation Notes on Compilation
- **Diablo Workflow**: Diablo's `EthereumWorkloadGenerator` and `SolanaWorkloadGenerator` attempt to compile contracts on-the-fly if `solc` or `solang` are available on the system.
- **Contender Workflow**: Contender typically expects pre-compiled bytecode in the scenario TOML or artifacts. We have now provided the pre-compiled bytecode directly in the scenario.

### Workflow
1.  **Artifact Extraction**: Located the Uber contract and workload YAML in the `minion-eurosys23` directory.
2.  **Code Analysis**: Analyzed `ContractUber.sol` to understand state initialization and the distance calculation loop.
3.  **Scenario Refinement**: Updated the Contender scenario to match the actual contract and function names.
4.  **Reporting**: Documented the mapping and implementation details.
