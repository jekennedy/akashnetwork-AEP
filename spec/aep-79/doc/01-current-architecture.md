# 01. Current architecture: Akash on Cosmos SDK

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-01 |
| Version | 0.9 draft, 2026-08-10 |
| Baseline | `akash-network/node` commit `096bff57`; API `pkg.akt.dev/go` v0.2.14 |
| Purpose | Compact behavioral baseline for the target implementation |

The live chain, not repository defaults, is authoritative. At kickoff, export all live parameters, balances, vesting, supply classes, IBC escrows, state sizes, and transaction/query mix (Q-19). Do not copy known defects listed below.

## End-to-end model

Tenants submit an SDL. Only its canonical hash and marketplace state are on chain; the provider daemon obtains and validates the manifest off chain. Identity is `(owner,dseq,gseq,oseq,bseq)`, with `bseq=0` in current accepted flows.

1. `x/deployment` creates a deployment and groups, funds escrow, and opens one market order per group.
2. Providers registered in `x/provider` bid. Matching checks price, resources, self-declared attributes, and `x/audit` attestations.
3. The tenant selects a bid. `x/market` creates a lease and streaming payment and closes losing bids.
4. `x/escrow` lazily accrues payment. Providers settle/withdraw; tenants top up or close.
5. Closing cascades escrow → market → deployment. A tenant-closed lease relists its group if the group remains open.

## Module behavior

| Module | State and behavior to understand | Target disposition |
|---|---|---|
| `x/deployment` | Deployment/group lifecycle. Creation accepts ACT only; group price must be ACT. Close/pause/start drives market and escrow hooks. | Preserve lifecycle and manifest hash; bounded per-deployment/group state. |
| `x/market` | Orders, bids, leases; max 20 bids/order; bid minimums 500,000 uAKT/uACT; reclamation `1h–720h`. Tenant selects winner. Losing collateral is refunded. | Preserve; asynchronous losing-bid reaps are allowed on Solana. Provider deletion becomes staged deregistration. |
| `x/escrow` | Account/payment stores, lazy accrual, FIFO depositors, authz-funded deposits, refunds that restore authorization, explicit overdrawn debt, ACT-first settlement and AKT fallback. | Preserve exactly, replacing authz with funded allowances. |
| `x/bme` | Queued AKT↔ACT swaps, BME vault, time/epoch execution, collateral ratio, circuit breakers, spreads, retries. | Preserve economic behavior; execute by crank/keeper. |
| `x/oracle` | One authorized Pyth CosmWasm source; staleness/deviation gating. | Replace with direct Pyth pull reads. |
| `x/epochs` | Schedules oracle and BME block handlers. | Replace with timestamp gates and permissionless execution. |
| `x/provider` | Provider host, attributes, contact; delete is unimplemented. | Add owner/operator keys, TLS SPKI, and staged deregistration. |
| `x/audit` | Auditor-signed provider attributes consumed by matching. | Preserve provenance and deletion/upsert semantics. |
| `x/cert` | On-chain x509 certificates and revocation. | Do not port; replace with registered keys and wallet-signed JWTs. |
| `x/wasm`, `awasm`, `contracts/` | CosmWasm exists to verify/post Pyth prices; awasm restricts allowed messages. | Do not port; targets read Pyth directly. |
| `x/take` | Dead/unwired fee code. Providers receive 100%. | Do not port unless governance makes a new decision. |

## Escrow semantics

Escrow is the highest-fidelity port requirement.

- Settlement is lazy: `accrued = rate × elapsed`; it runs only on close, overdrawn deposit, payment create/withdraw/close, or explicit keeper calls. There is no reliable per-block settlement caller in the repository.
- Current rates are 18-decimal `LegacyDec` values per block. Target rates are fixed-point per second, using the exact one-time conversion `rate_per_second = rate_per_block × 2/13`.
- Transfer boundaries truncate to integer micro-units; no outflow rounds up.
- Deposits record multiple contributors in FIFO order. Deduction and refunds use that order. `DepositAuthorization` grants may fund a deposit, and unused refunded value restores the originating grant's spend limit.
- When funds are exhausted, state records a negative/unsettled obligation and freezes the settlement point. Deposit-while-overdrawn settles before adding funds. The target may use explicit fields but must preserve the transition behavior.
- Normal drain order is ACT, then any configured settlement stablecoin at governed ACT parity. Under BME `halt_cr`, ACT debt may settle from AKT at a healthy oracle price; under `halt_oracle`, fallback is blocked.
- There is no active protocol take: providers receive 100% of accrued payment.
- Close order matters: settle, close payments/account, refund depositors, then market/deployment hooks. Target guarded reaps must make intermediate states safe and idempotent.

Core conservation rule per account:

`deposits = withdrawals + refunds + held funds + accrued/unsettled obligations`, subject only to documented truncation dust.

## BME semantics

BME converts AKT and ACT against oracle value and guards ACT backing.

- Requests escrow input immediately and execute FIFO in epoch batches. Default maximum is 50 records; failing records retry up to 3 times before cancellation/refund.
- Minimum ACT mint is 10 ACT. Default mint spread is 25 bps; settlement spread is 0 bps.
- Pending AKT is excluded from collateral available to back ACT.
- Collateral ratio:

  `CR = (vault_uakt - pending_uakt) × (AKT/USD ÷ ACT/USD) ÷ ACT_liability`

  Target liability includes issued ACT plus protocol-tracked escrowed ACT.
- Status thresholds are warning at `9500 bps` and CR halt at `9000 bps`. An unhealthy oracle produces `halt_oracle`.
- AKT→ACT minting stops at CR halt. ACT→AKT redemption remains allowed at CR halt but not oracle halt; this exit valve must not be removed.
- Execution order is burn input, update status/postcondition, then mint output. A batch must stop before it violates the halt threshold.
- Epoch timing, record caps, retry backoff, spreads, minimums, and vault funding are governance parameters. The target uses 65-second nominal epochs and permissionless execution.

Oracle defaults in the code baseline are 30-second staleness and 150 bps confidence/deviation bounds; an apparent 5-second oracle TWAP versus a 1-hour BME window conflict must be traced at kickoff (Q-41).

## Chain and token baseline

| Topic | Current value or behavior |
|---|---|
| AKT | `uakt`, 6 decimals, transferable, fee/staking token |
| ACT | `uact`, 6 decimals, `SendEnabled=false`, BME/market settlement asset |
| Block target | 6.5 seconds |
| Tx processing | Standard ante handler; unordered transactions enabled; no custom mempool or vote extensions |
| Validators | Up to 100; 14-day unbonding; 5% minimum commission |
| Slashing | 5% double-sign; downtime slash 0; 30,000-block window; short jail duration |
| Governance defaults | 2,500 AKT minimum deposit, 14-day deposit period, 3-day voting, 20% quorum |
| Fees | Source conflict: `0.0025` versus `0.025 uakt/gas`; live value must be exported |
| IBC | ICS-20 transfer plus v2 route; module/escrow balances require explicit S1 disposition |

Akash maintains forks of Cosmos SDK, CometBFT, and gogoproto. Eliminating that maintenance and the validator/consensus operating surface is a primary migration benefit.

## Off-chain dependencies

- `provider-services` contains Kubernetes/provider logic and direct chain integration. Keep its workload logic; insert a `pkg/chain` adapter.
- `pkg.akt.dev/go`, `@akashnetwork/chain-sdk`, CLI, Console, indexer, network metadata, snapshots, Hermes price feeder, wallets, explorers, and exchange integrations all depend on addresses, events, queries, or transaction formats.
- SDL/manifest serialization and hash generation are byte-sensitive and must remain cross-language compatible.
- Clients rely on the discovery/min-client-version contract; replace it with the target config registry.
- Event delivery is at-least-once. Indexers and provider daemons must deduplicate, survive replay/reorg, and reconcile against authoritative chain state.

## Do not port these defects

- Unpopulated or stale mint/consensus genesis and upgrade tables.
- `ContractDebugMode=true` in wasmd configuration.
- Conflicting minimum gas-price defaults.
- Incorrect ACT display denomination metadata.
- Unwired crisis invariants and dead `x/take`/wasm query paths.
- Provider deletion returning `NOTIMPLEMENTED`.
- Hard-coded Pyth/governance addresses without a governed config registry.
- Any source default that disagrees with exported `akashnet-2` state.

The target mappings are in [14](./14-appendix-protocol-mapping.md); normative target designs are [03](./03-solana-architecture.md) and [04](./04-ethereum-architecture.md).
