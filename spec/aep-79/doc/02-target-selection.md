# 02. Target selection

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-02 |
| Version | 0.9 draft, 2026-08-10 |
| Decision | D-01: choose Solana mainnet or one existing EVM L2 at Gate 0 |

## Why move

Akash gains little by continuing to fund and operate a sovereign consensus layer when its product is a compute marketplace. Moving to a shared chain should release bonded capital, retire validator/Cosmos-fork/oracle infrastructure, reduce transaction cost, improve wallet/stablecoin/exchange distribution, and widen the engineering/audit labor pool. It also gives up sovereign blockspace, validator revenue, native IBC, and control over consensus upgrades. Gate 0 must decide whether the trade is justified using current evidence.

## Target requirements

The selected chain must satisfy `REQ-GEN-001–020`:

| Area | Minimum evidence |
|---|---|
| Throughput and cost | ≥50 mixed marketplace tx/s sustained; ≥500 tx/s for 60-second bursts; median simple protocol tx ≤$0.01 and not >$0.05 under normal load; current/3×/10× cost model |
| Finality and liveness | Effective confirmation ≤5 seconds for marketplace UX; ≥99.5% liveness over the prior 24 months; documented degraded/finality tiers |
| Oracle and settlement | Production AKT/USD Pyth pull feed or equivalent updated <1 minute; native, deeply liquid stablecoin |
| Token and custody | AKT support path for top venues and at least two institutional custodians; non-transferable ACT and secp256k1 claim verification feasible |
| Governance and operations | Mature DAO, timelock, multisig, emergency pause, gas sponsorship/account abstraction, two independent indexing paths |
| Security and ecosystem | At least three credible audit firms and a deep developer pool; production migration precedent; no structurally unavoidable single operator, or a credible forced-inclusion/decentralization roadmap |

All fee, feature, support, and decentralization claims are volatile and must be reverified during Stage A.

## Options

| Option | Status | Assessment |
|---|---|---|
| A: Solana mainnet | Gate 0 candidate | Best raw fee/throughput and DePIN fit; native Pyth and Token-2022 support ACT. Risks: Token-2022 venue support, Go/provider integration, congestion, and Alpenglow timing. |
| B1: existing EVM L2 | Gate 0 candidate | Portable Solidity stack, mature audits/governance/claims/AA, strong exchange distribution. Host selected by Q-42 against stablecoin, oracle, fee, RPC, indexer, forced-inclusion, decentralization, and policy criteria. |
| B2: Arbitrum Orbit L3 | Non-default fallback | More control and AKT gas, but recreates sequencer/DA/ops burden and adds recurring cost/licensing/bridge complexity. Revisit only if B1 fees or throughput fail Q-06. |
| Ethereum L1 | Rejected | Fees and throughput do not fit a chatty marketplace. |
| SVM L2/network extension | Not a current candidate | Less mature operations, liquidity, and migration precedent than Solana mainnet. |
| Cosmos shared security | Superseded | Retains too much Cosmos/consensus operational burden and does not maximize distribution benefits. |
| Status quo | Baseline only | Avoids migration risk but retains bonded-capital, fork maintenance, limited distribution, and scaling costs. |

## Gate 0 evidence and process

Before target-specific production work, Stage A must publish evidence for `REQ-GEN-001–020`, with special focus on:

1. Token-2022 support across the relevant exchanges and custodians, including the legacy SPL fallback trigger.
2. Solana Alpenglow rollout and stability.
3. Written top-venue integration feedback on Solana versus EVM.
4. A measured provider-services prototype showing Solana integration effort relative to EVM.
5. EVM host-chain candidates against all B1 criteria, including fees, native stablecoin, Pyth, ERC-4337, automation, RPC/indexer redundancy, forced inclusion, and policy stability.
6. Mainnet-like claims gas/compute and paymaster economics for single-wallet, Ledger, and multisig paths.

Gate 0 sequence:

- Publish the verification report and scored comparison.
- Submit a non-binding `akashnet-2` signal proposal.
- Record community tally and Overclock leadership decision.
- Name the selected target and, for EVM, the host chain; update D-01/Q-42.
- Authorize only the selected Stage B implementation. Retain the alternate architecture as a documented fallback.

`REQ-GEN-001–020` also require the decision package to disclose what Akash gives up: sovereign consensus/control, validator staking revenue, IBC-native transfers, and the old-chain identity narrative. Communications must explain the end of staking, the claim/swap process, 90-day wind-down, and S1 point of no return.

Related: [03](./03-solana-architecture.md), [04](./04-ethereum-architecture.md), [10](./10-rollout-and-cutover.md), and the canonical decision log in [13](./13-open-questions-and-assumptions.md).
