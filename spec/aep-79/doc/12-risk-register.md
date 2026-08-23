# 12. Risk register

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-12 |
| Version | 0.9 draft, 2026-08-10 |
| Owner | Joint program office |

## Method

Likelihood and impact use Low=1, Medium=2, High=3; score=`L×I`. Likelihood anchors: Low <25%, Medium 25–60%, High >60% during kickoff→H+90d. Impact anchors: Low <1 month/<10% budget; Medium 1–3 months/10–25% or bounded recoverable harm; High >3 months/>25%, launch-constituency loss, program stop, or any unrecoverable fund/supply failure.

Score 9 is Critical, 6 High, 3–4 Medium, and 1–2 Low. Owners: `VEL` Vendor engineering, `VSL` Vendor security, `OPL` Overclock protocol, `OIL` Overclock infrastructure/ops, `OBD` ecosystem/BD, `OCL` community/governance, `LC` legal, `PMO` joint program office.

Review Medium+ monthly, all risks at G0–G5, and re-score within five business days when a trigger or linked assumption fires. Keep IDs stable; move retired/path-descoped rows to the killed-risk log.

## Master register

| ID | Path | L/I/S | Owner | Risk and primary mitigation | Trigger / contingency |
|---|---|---:|---|---|---|
| R-01 Claims exploit | Both | M/H/6 | VSL | Claims can mint migrated supply. Immutable attested roots, one-claim receipts, constrained mint, two audits, formal/property checks, velocity breaker, top bounty. | Conservation/velocity/audit alert → pause claims, patch against same roots, reconcile publicly. |
| R-02 Snapshot mismatch | Both | M/H/6 | VEL | Complex balance classes may violate conservation. Two independent exporters, exact roots, ≥3 dry runs, public explorer/challenge. | Any root/data diff → hold; before open replace candidate, after open pause and governance-fund correction. |
| R-03 Escrow parity loss | Both | M/H/6 | VEL | FIFO/overdrawn/time/fallback port may mispay. Mainnet golden replay, invariants, escrow-heavy audit. | Parity or chain/indexer drift → pause intake, upgrade, apply approved reimbursement. |
| R-04 BME/oracle manipulation | Both | M/H/6 | VSL | Thin-market/latency attacks can drain CR. 9500/9000 breaker, spread, minimum/caps, Pyth gates, economic red-team. | CR/queue-price anomaly → CR halt, allow healthy ACT exits, top up/tighten parameters. |
| R-05 Crank/keeper liveness | Both | M/M/4 | OIL | Automation failure delays settle/BME/residuals. Flat tips, redundant executors, keeper SLAs, lag alarms. | Lag/tip depletion → manual execution and governance budget/tip adjustment. |
| R-06 Sponsor drain | Both | M/M/4 | OIL | Sybil spam drains relayer/paymaster. Allow-list, quotas, caps, hot-float isolation, anomaly breaker. | Spend anomaly → stop sponsorship, tighten/refill, use user-paid fallback. |
| R-07 Alpenglow instability | SOL | M/M/4 | PMO | Consensus transition near launch may degrade Solana. Stability buffer and Alpenglow cluster soak; avoid fixed slot assumptions. | Activation/incidents → delay C by governance; EVM remains available before G0. |
| R-08 Solana congestion | SOL | M/M/4 | VEL | Fee/landing spikes harm bidding. Per-entity sharding, bounded priority fees/retries, congestion tests. | p95 fee/latency breach → subsidize critical paths and defer low urgency. |
| R-09 Token-2022 support | SOL | M/M/4 | OBD | Venue/custody gaps fragment liquidity. Q-05 survey and early tests; predesigned legacy SPL fallback. | Top-venue threshold fails G1 → one canonical legacy SPL mint. |
| R-10 SOL rent cost | SOL | M/L/2 | VEL | SOL price raises entity deposits. Small close/refund accounts; monitor effective cost. | Threshold breach → compression or governed rent subsidy. |
| R-11 EVM host/Orbit policy | EVM | M/M/4 | PMO | Host fees/policy or Orbit terms may shift. Gate 0 policy scoring, portable suite, documented alternatives. | Material announcement → retarget before G1 or invoke Q-06/B2/redeploy. |
| R-12 Exchange delay | Both | H/M/6 | OBD | Per-venue swaps can lag months/years. Start 3–6 months early, same-ticker packs, volume gate, dedicated owner. | Commitments below gate → publish venue status, seed liquidity, extend support. |
| R-13 Stranded IBC AKT | Both | M/M/4 | OCL | Counterparty vouchers miss S1. Return-home campaign, counterparty inventory, bounded redemption reserve. | IBC-out balance not declining → extend outreach/window by governance. |
| R-14 Validator attrition | Both | M/H/6 | OIL | Post-S1 value loss/zero downtime slash may halt wind-down. Fund Q-13, >2/3 commitments, weekly monitoring, backstops. | Power/missed blocks → restart; worst case early S2 from last good height. |
| R-15 Provider no-show | Both | M/H/6 | OBD | Re-registration friction leaves no supply. Council preview, one-command tool, testnet, incentives, white-glove top cohort. | Capacity below rollout threshold → boost incentives and seed Overclock capacity. |
| R-16 Tenant interruption | Both | M/M/4 | OBD | Workloads may not redeploy before H. Repeated notices, guided redeploy, provider support, no early forced close. | Active old leases at C+45d → escalated help/credits; S2 refunds at H. |
| R-17 Demand pause | Both | H/M/6 | OBD | Customers defer during uncertainty. Launch target before C, credits, direct outreach, publish parity/uptime. | Deployment rate falls → treasury-funded demand incentives. |
| R-18 Competitor capture | Both | H/L/3 | OBD | Migration consumes roadmap attention. Keep old chain supported to C, differentiated positioning, post-launch roadmap. | Competitor/churn signals → targeted win-back. |
| R-19 Governance rejection | Both | M/H/6 | OCL | Migration ends validator role and may fail/fragment vote. Signal before G0, address validator program, public concrete plan. | Weak signal/sentiment → revise/revote or stop at G0. |
| R-20 Old-chain fork | Both | L/H/3 | OCL | A faction may continue `akashnet-2` and claim AKT identity. Venue alignment, trademark preparation, canonical claims. | Fork coordination → exchange advisories and legal brand action. |
| R-21 Post-S1 spam | Both | M/L/2 | OPL | Claim-inert gas token enables cheap state spam. Sunset allow-list, fee raise, growth monitoring. | Tx/state spike → emergency parameter increase. |
| R-22 Multisig compromise | Both | L/H/3 | OPL | Upgrade/treasury signer loss or collusion. Hardware 4-of-7+, org/jurisdiction spread, timelock, payload review. | Key incident/unknown queue → cancel, rotate, incident response. |
| R-23 Early governance capture | Both | L/H/3 | OPL | Low turnout/unclaimed supply enables hostile control. Quorum floors, timelock, council cancel/pause, staged authority, spend caps. | Voting concentration/hostile queue → cancel/pause and rotate authority. |
| R-24 Realms maintenance | SOL | M/M/4 | VEL | Thin upstream maintenance may force a fork. Q-11 gap analysis and minimal-fork budget; Squads protects upgrade path. | Missing feature/release stagnation → minimal fork or alternate stack at G1. |
| R-25 Indexer/RPC lock-in | Both | M/M/4 | OIL | Provider/framework pivots can break Console data. Standard ingestion seam, vendored/pinned framework, two RPCs, self-host plan. | Roadmap/cost/lag breach → swap vendor or degraded public-RPC mode. |
| R-26 Pyth degradation | Both | M/M/4 | OPL | Single feed can halt BME/fallback. Publisher engagement, health gates, integration-ready secondary (Q-24). | Age/confidence/publisher decline → oracle halt then governance switch. |
| R-27 Claims regulation | Both | M/H/6 | LC | Global portal may face MiCA/sanctions rules. Legal review before G2; non-custodial protocol and separable UI. | Counsel/regulator action → geo-policy; CLI/direct claim remains available where lawful. |
| R-28 Key-person loss | Both | M/M/4 | PMO | Current/target knowledge is concentrated. ≥2 Overclock FTE, pairing/rotation, documentation, Vendor bench. | Attrition/single owner → backfill and gate rebaseline. |
| R-29 Scope/budget creep | Both | H/M/6 | PMO | Large dual-path specification invites churn. G0 retires a path; frozen IDs, change board, milestone payments. | CR/earned-value variance → descope, reserve, or steering acceptance. |
| R-30 Cosmos-fork decay | Both | M/M/4 | OPL | Security bug may hit understaffed old forks. Named maintainer through H, security-only policy, advisory monitoring. | Upstream CVE/divergence → emergency old-chain patch or accelerated halt. |
| R-31 Audit cascade | Both | H/M/6 | VSL | Novel high-value code likely yields stacked findings. Shift-left review, tranche freezes, prebooked re-audit. | Critical/high burn-down slips → reduce audited launch scope or move C. |
| R-32 Manifest hash drift | Both | M/M/4 | VEL | Cross-language serialization may diverge late. Mainnet manifest vectors across provider/Console/SDK from G1. | Conformance/rejection spike → off-chain hotfix or temporary dual-read shim. |
| R-33 Unclaimed-funds law | Both | L/M/2 | LC | Sweeping after two years may violate abandoned-property rules. Counsel before G2, publish policy, conservative extension. | Counsel/claim telemetry → extend and segregate contested funds. |
| R-34 Claim phishing | Both | H/M/6 | OCL | Fake portals can redirect signed recipient entitlement. One canonical domain, payload-bound recipient, wallet allow-lists, takedown monitoring. | Lookalike/reports → rapid takedown/blocklists and incident support/comms. |
| R-35 Residual disputes | Both | M/M/4 | VEL | Off-chain weekly roots may be challenged or delayed. Independent exact roots, public inputs/tool, 48h evidence-based objection. | Reproducible mismatch or >96h → stop one cycle, supersede root, absorb next cycle. |

## Priority and gate rules

Current High risks: R-01–04, R-12, R-14–15, R-17, R-19, R-27, R-29, R-31, and R-34. Highest priority by irreversibility/proximity is R-01 claims, R-02 snapshot, R-03 escrow, R-04 BME, then R-19 governance, R-15 providers, R-14 validators, R-12 exchanges, R-17 demand, and R-29 scope.

Acceptance authority:

- Critical 9: cannot be accepted; steering treatment within five business days and weekly reporting.
- High 6: mitigate; proceeding without more mitigation needs a recorded steering decision in [13](./13-open-questions-and-assumptions.md).
- Medium 3–4: PMO may mitigate or accept with rationale.
- Low 1–2: owner accepts and monitors.

No gate may pass with a score-9 risk, an unfunded/unaccepted High risk, or a fired trigger without re-score. Cutover gates also enforce target stability, venue coverage, validator commitments, and provider capacity.

Proactive treatment belongs in baseline workstream budgets. High/Critical owners maintain expected-cost estimates. Their total must fit the remaining contingency reserve; otherwise rebaseline before the next gate. A single draw >25% of remaining reserve needs steering approval. Record each draw against its R-ID.
