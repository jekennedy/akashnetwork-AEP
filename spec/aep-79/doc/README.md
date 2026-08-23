# AEP-79: Akash chain migration

| Field | Value |
|---|---|
| Status | Draft v0.9 |
| Baseline date | 2026-08-10 |
| Owner | Overclock Labs |

AEP-79 specifies how to move the Akash marketplace from the sovereign Cosmos SDK chain `akashnet-2` to either Solana mainnet or an existing EVM L2. Gate 0 selects one target; only that path is built. The migration preserves marketplace and economic behavior while retiring Akash-operated consensus, staking, IBC, CosmWasm oracle plumbing, and the on-chain x509 registry.

## Document map

| Doc | Use it for |
|---|---|
| [00](./00-executive-summary.md) | Program purpose, migration shape, effort, and major risks |
| [01](./01-current-architecture.md) | Current Cosmos behavior that must be preserved or deliberately retired |
| [02](./02-target-selection.md) | Target requirements and Gate 0 decision process |
| [03](./03-solana-architecture.md) | Solana program, account, token, and runtime design |
| [04](./04-ethereum-architecture.md) | EVM contract design and the optional Orbit variant |
| [05](./05-token-migration.md) | Supply accounting, claims, vesting, exchanges, and emissions |
| [06](./06-state-and-data-migration.md) | Export pipeline, residual accounting, old-chain sunset, and archives |
| [07](./07-offchain-and-clients.md) | Provider daemon, auth, indexer, SDK, CLI, Console, and portal |
| [08](./08-security-and-audits.md) | Threat model, authority controls, audits, invariants, and operations |
| [09](./09-testing-and-verification.md) | Parity harness, test layers, rehearsals, performance, and CI |
| [10](./10-rollout-and-cutover.md) | Phases, gates, governance sequence, cutover, and contingencies |
| [11](./11-scope-of-work.md) | Vendor workstreams, milestones, staffing, and commercial controls |
| [12](./12-risk-register.md) | Risk scores, owners, triggers, mitigation, and acceptance rules |
| [13](./13-open-questions-and-assumptions.md) | Canonical decisions, assumptions, open questions, and volatile facts |
| [14](./14-appendix-protocol-mapping.md) | Cosmos-to-Solana/EVM implementation lookup |

Suggested reading: decision-makers `00 → 02 → 10 → 12`; protocol engineers `01 → 13 → 03/04 → 05 → 06 → 14`; security/test teams `08 → 09 → 10`; vendors `00 → 11`, then the technical documents for their workstream.

## Conventions

- `MUST`, `SHALL`, `SHOULD`, and `MAY` use RFC 2119 meanings.
- Stable identifiers: `REQ-*` requirement, `D-*` decision, `A-*` assumption, `Q-*` open question, `R-*` risk.
- Events: `C` = cutover and S1 snapshot; `H` = final old-chain block, normally `C+90d`; `S2` = final residual snapshot at `H`.
- Gates: `G0` target selection, `G1` design freeze, `G2` code complete/testnet/audits started, `G3` launch readiness, `G4` S1 verified, `G5` close-out.
- Calendar dates live in the program plan; these documents define event-relative sequencing.
- Values marked `TO-VERIFY` and all ecosystem facts must be rechecked at Vendor kickoff.

Changes use pull requests. Do not reuse identifiers; record changed or withdrawn items in [13](./13-open-questions-and-assumptions.md). After G1, a normative change requires impact analysis for implementation, tests, audits, schedule, and risk.
