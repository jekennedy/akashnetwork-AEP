---
aep: 79
title: "Akash on Shared Security"
author: Greg Osuri (@gosuri)
status: Final
type: Standard
category: Core
created: 2025-12-15
updated: 2026-08-11
estimated-completion: 2026-12-31
roadmap: major
---

> **Current migration specification:** See [doc/README.md](./doc/README.md). The execution-level migration documents supersede this original RFP where they conflict.

## Abstract

Akash Network plans to leave its sovereign Cosmos SDK chain and inherit security from a shared chain. The goals are to release capital currently bonded for consensus, retire validator and L1 maintenance overhead, reduce costs, and let engineering focus on the decentralized-compute marketplace.

The original RFP invited established Layer 1 ecosystems to propose a shared-security model. A proposal had to explain its security, integration and migration path, scalability, governance, economics, legal considerations, liquidity, and ecosystem support. The current candidate targets and selection process are now defined in [doc/02-target-selection.md](./doc/02-target-selection.md).

## Motivation and objectives

Akash coordinates tenants seeking compute with providers offering CPU, GPU, storage, and networking. Deployments, orders, bids, leases, provider records, and escrow are represented on chain; workloads run in provider infrastructure.

Running a dedicated chain requires substantial AKT staking, validator coordination, upgrades, infrastructure, and security maintenance. A shared chain should:

1. Replace the continuous AKT security lock with usage-based transaction and infrastructure costs.
2. Remove Akash-operated consensus and validator-set maintenance.
3. Preserve open participation, transparent governance, and application behavior.
4. Support high marketplace throughput with predictable fees and finality.
5. Provide practical token, stablecoin, wallet, exchange, custody, oracle, and indexing infrastructure.

Akash accepts the corresponding trade-off: consensus, blockspace, and some infrastructure policy are inherited from the selected network rather than controlled by an Akash validator set.

## Proposal evaluation areas

The original RFP requested evidence in the following areas:

| Area | Required evidence |
|---|---|
| Security | Consensus/shared-security mechanism, validator incentives, slashing, decentralization, liveness, and recovery |
| Integration | Execution environment, Cosmos/IBC or equivalent interoperability, required code changes, governance steps, and migration schedule |
| Performance | Sustained and burst throughput, finality, capacity constraints, fees under normal and stressed demand, and scaling roadmap |
| Governance | Upgrade/parameter authority, transparency, community participation, onboarding support, grants, and long-term collaboration |
| Economics and legal | Adoption and operating costs, staking or native-token dependencies, AKT implications, fee assets, and regulatory constraints |
| Ecosystem | CEX/DEX liquidity and market depth, stablecoin/custody/wallet support, investors, institutional adoption, and growth funding |

The current Gate 0 requirements, including quantitative performance and cost thresholds, are maintained in [doc/02-target-selection.md](./doc/02-target-selection.md).

## Current Akash architecture

The current implementation is a Go-based Cosmos SDK application. Its relevant state and behavior are:

| Component | Responsibility |
|---|---|
| Deployment | Workload deployment and group lifecycle; anchors the off-chain SDL/manifest hash |
| Market | Orders, provider bids, tenant-selected leases, and reclamation |
| Escrow | Deposits, lazy streaming settlement, withdrawals, and refunds |
| BME/oracle | AKT↔ACT conversion, collateral-ratio protection, and AKT/USD pricing |
| Provider/audit | Provider registration, attributes, and auditor attestations |
| Cosmos infrastructure | Accounts, bank, staking, distribution, slashing, governance, IBC, upgrades, and consensus |

The provider stack performs the actual workload orchestration and talks to the chain through APIs and events. The migration preserves marketplace and escrow semantics while replacing the Cosmos runtime, consensus-dependent handlers, chain access, identity, and indexing infrastructure. See [doc/01-current-architecture.md](./doc/01-current-architecture.md) for the behavioral baseline and [doc/14-appendix-protocol-mapping.md](./doc/14-appendix-protocol-mapping.md) for target mappings.

## Current migration direction

Gate 0 selects one of two implementation paths:

- Solana mainnet.
- An existing general-purpose EVM L2, with the host selected through the Gate 0 evidence process.

The target rebuilds the marketplace rather than transporting live contract state. AKT/ACT value moves through an S1 snapshot, weekly wind-down distributions, and a final S2 snapshot; existing leases wind down on the old chain for 90 days. There is no persistent migration bridge. Full architecture, security, testing, cutover, and vendor scope are indexed in [doc/README.md](./doc/README.md).

## Submission

Overclock Labs coordinates evaluation. Interested parties should contact Greg Osuri on [X](https://x.com/gregosuri) or [Telegram](https://t.me/gregosuri).

## Copyright

Licensed under [Apache 2.0](https://www.apache.org/licenses/LICENSE-2.0).
