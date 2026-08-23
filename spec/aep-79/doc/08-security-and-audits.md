# 08. Security and audits

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-08 |
| Version | 0.9 draft, 2026-08-10 |
| Requirement family | `REQ-SEC-001–077` |

## Changed trust model

The migration removes validator/slashing, IBC, Akash-hosted CosmWasm, and a single authorized oracle pusher. It adds a more concentrated upgrade authority, a claims system capable of minting most migrated supply, crank/keeper liveness, fee sponsorship, public-chain ordering, and inherited L1/L2 outages/censorship.

Unlike a sovereign-chain upgrade, the target cannot halt consensus and rewrite state. Every fund-holding component therefore needs intake-only pause, a rehearsed upgrade/recovery path, independent state archives, live invariant monitoring, and on-chain conservation assertions where affordable (`REQ-SEC-001–004`).

## Component controls

### Claims (`REQ-SEC-005–012`)

- Registered roots are immutable and append-only; disable root registration permanently after final S2/redemption roots.
- Before activation, at least three independent parties recompute each root and publish signed attestations.
- Enforce one claim per leaf/root on chain.
- Verify the exact domain-separated Cosmos digest, low-S secp256k1, and address derived from the verified pubkey. Enforce recorded Cosmos multisig threshold and distinct signers.
- Bind root, target chain, claims address, and recipient to defeat replay.
- Segregate claims mint/balance from all other protocol funds.
- Auto-pause claims when a governance-set 24-hour velocity threshold (Q-29) is exceeded.

### Escrow (`REQ-SEC-013–016`)

All outflows truncate; no rounding up. Bound depositors at 16 initially and prove worst-case refund fits one transaction. Preserve settle-before-deposit, frozen timestamp/debt semantics while overdrawn, FIFO deduction, and exact allowance restoration to the originating granter. Fuzz boundary transitions and assert conservation.

### BME (`REQ-SEC-017–021`)

Every request/execution validates Pyth staleness/confidence. Enforce minimum 10 ACT-equivalent, batch/account/epoch caps, one execution-time price per batch, and max three attempts. The vault has one outflow path and no admin withdrawal. Recheck CR during mint execution. Keep ACT→AKT exits open at CR halt but closed at oracle halt; batch/backoff caps and healthy pricing bound the run dynamic.

### Marketplace and provider identity (`REQ-SEC-022–026`)

Prevent free exhaustion of the 20 bid slots using the Q-08-ratified collateral/fee/forfeiture policy. Because current losing-bid collateral is refunded, any non-refundable fee or forfeiture is an intentional economic delta and must be recorded in [13](./13-open-questions-and-assumptions.md), modeled, and parity-tested. Preserve auditor provenance for attributes. Provider signing-key rotation is owner-authorized, fully evented, queryable, immediately checked by clients, and alerts the provider operator.

### Governance, relayer, and read path (`REQ-SEC-027–035`)

All upgrades, mints, treasury movements, oracle/config changes, and privileged roles pass through multisig/DAO plus public timelock. Security council may pause/cancel, never execute arbitrary value movement. Relayers/paymasters allow-list methods, validate full transactions, rate-limit identities/IPs, cap per-tx/daily/global spend, isolate hot float, and provide user-paid fallback. Before signing any economic action, provider/Console clients confirm authoritative chain state; continuously sample indexer results against direct RPC.

## Platform-specific controls

**Solana (`REQ-SEC-036–040`).** Validate owner, signer/writable flags, discriminator, distinct mutable accounts, checked casts, and overflow checks. Use fixed-width/collision-free PDA seeds; pin CPI targets and signer scopes. Closure must prevent account resurrection. Publish frozen canonical ALTs and have clients verify their contents.

**EVM (`REQ-SEC-041–044`).** Use CEI and reentrancy guards around every token/protocol call; model restricted ACT as reentrant. Use permit or exact allowance, never standing unlimited approval. Enforce ERC-7201 layout compatibility. Disable implementation initializers and deploy/initialize proxies atomically.

## Ordering and MEV

No economic outcome may depend on transaction priority: the tenant selects the bid, claims have no first-come allocation, and each BME batch receives one execution-time price. Bound settle-timing extraction with aggregated prices, staleness/confidence limits, and quantified maximum value transfer. Crank/keeper tips are flat, never proportional to value (`REQ-SEC-045–047`).

## Keys, timelocks, and pause

Launch baseline (`REQ-SEC-048–055`):

- Upgrade authority is at least 4-of-7; no organization controls more than two seats; at least three jurisdictions.
- All signer keys are hardware-backed, environment-separated, published by organization/role, and quarterly canary-tested.
- Replace a departed/compromised signer within seven days; rehearse multi-signer loss and emergency re-key.
- Solana uses Squads + Realms; EVM uses Safe + `AkashTimelock`. Token mint roles belong to protocol programs/contracts, never humans directly.

| Operation | Minimum delay / approval |
|---|---|
| Routine bounded parameter | 48h, multisig at launch then DAO |
| Program/contract upgrade | 7d plus public diff/audit note |
| Weekly residual root | 48h plus independent attestation |
| Emergency security patch | 24h, 6-of-7, disclose within 72h |
| Guardian intake pause | Immediate, 2-of-7 sub-quorum |
| Component full stop | 4-of-7; auto-expires within 72h unless governance renews |

Intake pause may block new deployments/bids/deposits/swaps/emissions, but never settlement, withdrawal, refund, close, reclamation completion, or already queued safe exits. Claims pause only through the stricter full-stop/velocity-breaker path.

Authority sunset (`REQ-SEC-056–057`): launch multisig+timelock → DAO as sole proposer after S2 and two clean quarters → governance vote to revoke claims and escrow-core upgradeability after the Q-28 burn-in (default proposal 12 months). Once criteria are met, complete or explicitly defer Phase 2 within 90 days.

## Audit program

| Tranche | Scope | Required review | Launch gate |
|---|---|---|---|
| T1 tokens/claims | AKT/ACT, claims, Reserve/residual roots, signature/multisig verification, governance wiring | Two independent full-scope firms | Before any real-value token/claims deployment |
| T2 marketplace/economics | Escrow, BME, market/deployment, provider/audit, config, emissions | Two independent firms, economic review, public competition | Before marketplace mainnet |
| T3 off-chain/ops | Signer sidecar, relayer/paymaster, portal backend, monitors, ceremonies/runbooks | Code audit plus infrastructure pentest | Before C |

All critical/high findings must be fixed and reverified by the issuing firm before G3. Any material fund-flow diff after audit receives an original-firm delta review. Publish reports within 30 days of deployed fixes. Procure firms based on current expertise/availability rather than the examples in the earlier draft (`REQ-SEC-058–062`).

## Invariants

| ID | Invariant | Enforcement |
|---|---|---|
| I-1 | Escrow deposits = withdrawals + refunds + held/accrued/debt | Property test + monitor |
| I-2 | No negative balance except explicit internally consistent overdrawn debt | Property test + monitor |
| I-3 | Claimed/root total never exceeded; leaf consumed once | On-chain assertion + monitor |
| I-4 | No ACT mint executes below 9000-bps CR halt | On-chain assertion + monitor |
| I-5 | Refunds follow depositor FIFO | Property test |
| I-6 | ACT changes only through protocol mint/burn paths | By construction + monitor |
| I-7 | BME vault has exactly one outflow path | Review + monitor |

Encode I-1–I-7 as machine-checkable properties; formally or semi-formally verify claims and escrow arithmetic where practical (`REQ-SEC-063–064`).

## Operations

Before launch, run independent-source monitors for AKT/supply and escrow conservation, BME CR (warn 9700, page 9500), claims velocity, every authority/timelock/mint event, large withdrawals, crank/keeper lag, Pyth age/confidence, indexer divergence/head lag, relayer spend, and every pause/status change (`REQ-SEC-065–067`).

SEV-1 means active exploit/invariant failure: war room within 15 minutes, landed pause decision/transaction within 30 minutes, public acknowledgment within two hours. SEV-2 receives war room within one hour and mitigation plan within 24 hours. Maintain 24/7 pause-capable coverage during launches, C/S1, weekly roots, H/S2, upgrades, and Solana Alpenglow activation. Publish SEV-1/2 postmortems within 14 days. Warn tenants at ≤72 hours escrow runway (`REQ-SEC-068–071`).

Fund an Immunefi-class bounty at least two weeks before mainnet through the two-year claim window. Indicative maximums: critical $250k–$1m, high $50k–$100k, medium $10k–$25k, low $1k–$5k, capped at 10% of demonstrably at-risk funds (`REQ-SEC-072`).

Solana programs require published verifiable-build hashes; EVM contracts require explorer and Sourcify verification. Pin lockfiles/toolchains, audit dependencies, and sign releases (`REQ-SEC-073–075`).

## No bridge

Do not deploy AKT/ACT lock-mint or burn-mint bridging beyond migration claims and emissions. Any later multichain system requires burn/mint, at least three independent verifiers/operators, per-chain rate limits based on organic volume, and a dedicated security review/audit (`REQ-SEC-076–077`).
