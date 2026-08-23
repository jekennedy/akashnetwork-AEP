# 14. Protocol mapping: Cosmos → Solana → EVM

| Field | Value |
|---|---|
| Doc ID | AKASH-MIG-14 |
| Version | 0.9 draft, 2026-08-10 |
| Status | Informative implementation lookup; docs 03/04 govern conflicts |
| Baseline | `akash-network/node` `096bff57`, `pkg.akt.dev/go` v0.2.14 |

Amounts are 6-decimal micro-units. Solana identifiers are program instructions and PDAs; EVM identifiers are contract functions/storage. Full behavioral context is [01](./01-current-architecture.md).

## Transaction mapping

| Cosmos message | Solana | EVM | Required behavior / delta |
|---|---|---|---|
| `MsgCreateDeployment` | `akash-deployment.create_deployment` | `DeploymentRegistry.createDeployment` | ACT-only create deposit and group price; create escrow and order(s); target assigns monotonic dseq. |
| `MsgUpdateDeployment` | `update_deployment` | `updateDeployment` | Active only; replace different canonical SDL/manifest hash. |
| `MsgCloseDeployment` | `close_deployment` + guarded reaps | `closeDeployment` | Settle/refund and terminal cascade; Solana may split child cleanup. |
| `MsgCloseGroup` | `close_group` | `closeGroup` | Close group and active order/bid/lease/payment. |
| `MsgPauseGroup` / `MsgStartGroup` | `pause_group` / `start_group` | `pauseGroup` / `startGroup` | Pause closes active market state; start creates next oseq. |
| Deployment `MsgUpdateParams` | `akash-config.set_deployment_params` | `AkashConfig.setDeploymentParams` | Governance+timelock only. |
| `MsgCreateBid` | `akash-market.create_bid` | `Marketplace.createBid` | Match price/resources/provider/audit attrs; max 20; `bseq=0`; collateral/Q-08 policy. |
| `MsgCloseBid` | `close_bid` | `closeBid` | Provider-owned; active lease requires completed `1h–720h` reclamation. |
| `MsgCreateLease` | `create_lease`, then `close_lost_bid` reaps | `createLease` | Tenant selects winner; payment rate from bid; losing collateral refunded. |
| `MsgCloseLease` | `close_lease` | `closeLease` | Tenant close relists if group stays open. |
| `MsgWithdrawLease` | `withdraw_lease`→escrow | `withdrawLease` | Lazy settle, provider receives 100%. |
| `MsgLeaseStartReclaim` | `lease_start_reclaim` | `startLeaseReclaim` | Wall-clock deadline within governed bounds. |
| Market `MsgUpdateParams` | config setter | config setter | Governance+timelock only. |
| `MsgAccountDeposit` | `akash-escrow.account_deposit` | `EscrowVault.accountDeposit` | Balance or funded allowance; settle first if overdrawn; ACT burns into logical escrow. |
| `MsgBurnMint` / `MsgMintACT` / `MsgBurnACT` | `akash-bme.request_burn_mint` / wrappers | `BurnMintEscrow.requestBurnMint` / wrappers | Queue input, execution-time price. ACT→AKT allowed at CR halt, blocked at oracle halt. |
| `MsgFundVault` | `akash-bme.fund_vault` | `BurnMintEscrow.fundVault` | Governance one-way funding; no admin outflow. |
| BME `MsgUpdateParams` | config setter | config setter | Convert epochs/limits to time-based config. |
| Oracle `MsgAddPriceEntry` | Pyth receiver update + direct read | `IPyth.updatePriceFeeds` + direct read | Retire authorized CosmWasm pusher after H; validate feed/age/confidence. |
| `MsgCreateProvider` / update | provider-registry create/update | `ProviderRegistry.createProvider` / update | Add JWT/operator keys, TLS SPKI, and optional collateral. Providers re-register. |
| `MsgDeleteProvider` | deactivate/finalize close | `beginDeregister` / `finalizeDeregister` | Replaces current unimplemented delete; require no live entities and delay. |
| `MsgSignProviderAttributes` / delete | audit sign/delete | `AuditRegistry.signProviderAttributes` / delete | Preserve `(provider,auditor)` provenance. |
| Cert create/revoke | Not ported | Not ported | Wallet JWT + provider registry replaces x509. |
| awasm/wasmd store/instantiate/execute | Not ported | Not ported | Direct Pyth pull replaces CosmWasm chain. |

### Stock Cosmos messages

| Cosmos | Target equivalent |
|---|---|
| Bank send AKT | Token-2022 `transfer_checked` or `AKT.transfer/transferFrom` |
| Bank send ACT | Disallowed; Token-2022 NonTransferable or restricted ERC-20 |
| Staking/delegation/rewards | No successor; S1 makes stake/rewards-through-C liquid |
| Gov submit/vote/deposit | Realms proposal/vote or `AkashGovernor`; no separate deposit phase |
| Authz `DepositAuthorization` | Funded allowance PDA or `EscrowVault` allowance; restore on refund |
| Generic authz | No general protocol successor; optional wallet/session delegation |
| Feegrant | Operated Solana fee payer or EVM ERC-4337 paymaster |
| ICS-20 transfer | No protocol bridge; return-home/redemption migration process only |

New target-only entrypoints include permissionless settle/reaps/queue execution, provider key rotation/finalization, claim/root management, vesting release, emissions execution, config registry, and pause/authority transition.

## Query and state mapping

All current deployment/group/order/bid/lease/escrow/payment/BME/provider/audit/parameter queries map to direct PDA/program-account reads plus the canonical indexer on Solana, or contract view calls plus the indexer on EVM. Current pagination/search/history queries become indexer APIs because terminal target state is deleted. `/v1` retains archive shapes; `/v2` uses target-native address/hash/finality and a `chain_id`.

| Cosmos state | Solana | EVM |
|---|---|---|
| Deployment/group | `Deployment`/`Group` PDA | mappings/structs in `DeploymentRegistry` |
| Order/bid/lease | per-entity PDA | mappings/structs in `Marketplace` |
| Escrow account/payment | `EscrowAccount`/`Payment` PDA and token vaults | `EscrowVault` mappings and held tokens |
| Authz deposit grant | funded `Allowance` + allowance vault | funded allowance mapping/vault accounting |
| BME vault/status/queue | BME state, vault, one PDA/request | `BurnMintEscrow` storage and queue records |
| Oracle prices/epochs | Pyth update accounts + timestamp gates | Pyth adapter + timestamps |
| Provider/audit | Provider and attestation PDAs | registry mappings |
| Cert/CosmWasm/IBC/staking | Archive only, no target state | Same |

Target state is bounded working state. Events carry full identifiers and amounts and are append-only history. Solana closes child-first and refunds the recorded rent payer; EVM deletes terminal structs after emitting the event when no later flow depends on them.

## Parameter mapping

| Current parameter | Target rule |
|---|---|
| Deployment minimum deposit | Governed per supported denom; create remains ACT-only |
| Group/resource/attribute bounds | 8 groups, 4 resource units, 24 attrs, 4 auditors; validate Q-38 |
| Bid deposits / max bids | Initial 500,000 uAKT/uACT and 20 bids; Q-08 controls economic anti-spam |
| Reclamation | 1h minimum, 720h maximum |
| Escrow depositor bound | Governed, initial 16 |
| BME epoch | Timestamp, nominal 65s; backoff up to 93,600s |
| BME batch/retry/minimum | 50 records, 3 attempts, 10 ACT-equivalent, plus epoch/account volume caps |
| BME warning/halt | 9500/9000 bps; mint spread 25 bps, settle spread 0 initially |
| Oracle | Default ≤30s age and ≤150 bps confidence; EMA for CR, spot for execution |
| Staking/slashing/mint | Retired; emissions schedule replaces inflation |
| Cosmos governance | Target DAO thresholds/timelocks per 03/04/08; EVM launch quorum proposal 10% |
| Cosmos gas floor/block gas | No direct mapping; target fee/compute budgets and sponsorship apply |

Live `akashnet-2` values override source defaults and must be exported under Q-19.

## Module-account disposition

| Cosmos account | Target |
|---|---|
| Escrow | Solana per-account token vaults/logical ACT or EVM pooled contract with per-account attribution; S1 balance enters Reserve |
| BME | Program/contract AKT vault and ACT mint/burn authority; seed from S1 Reserve after CR check |
| Fee collector | No Akash equivalent; fees pay Solana validators or EVM host/sequencer |
| Distribution/community pool | DAO treasury/Reserve disposition |
| Mint | Claims authority initially, then hard-capped emissions minter |
| Bonded/not-bonded pools | Liquid S1 claims; no target staking pools |
| Gov | Realms/Squads or Governor/Timelock; live deposits resolved in S1 ledger |
| IBC transfer/escrows | Reserved/redemption treatment; no persistent target module |

EVM cannot prevent a bare ERC-20 transfer to a contract address; internal accounting ignores unattributed funds and a governed, monitored sweep must never affect attributed balances.

## Identity, units, and time

| Cosmos | Solana | EVM | Rule |
|---|---|---|---|
| `akash1…` secp256k1 | 32-byte base58 Ed25519 key | 20-byte EIP-55 address | No conversion; old signature binds target recipient |
| `uakt`, 6 decimals | AKT base unit | AKT base unit | Exact 1:1 micro-unit |
| `uact`, 6 decimals, send-disabled | Token-2022 NonTransferable | Restricted ACT ERC-20 | Protocol mint/burn only |
| `LegacyDec`, 18 digits | `u128 ×1e18` | `uint256 WAD` | Checked fixed-point math |
| Pyth USD exponent | Normalize once | Normalize once | Shared price library; no floating point |
| dseq | per-tenant counter | `nextDseq[owner]` | Opaque monotonic; namespace by chain |
| gseq/oseq | same u32 semantics | same | oseq increments on relist/start |
| bseq | constant 0 API field | constant 0 | Not in target primary key |
| block height clock | `Clock.unix_timestamp` | `block.timestamp` | No slot/block economic arithmetic |
| per-block rate | per-second `×1e18` | same | S1 conversion exactly `×2/13` |
| unordered tx sequence | recent blockhash/durable nonce | account nonce/AA lanes | Provider clients queue/retry per target |

Transaction hashes and explorer URLs change: Cosmos uppercase SHA-256 hex, Solana base58 signature, EVM `0x` Keccak hash.

## Error compatibility

Clients must preserve stable semantic classes even when target numeric/selectors differ:

| Family | Required target distinctions |
|---|---|
| Escrow | invalid deposit/allowance scope, account/payment missing or closed, overdrawn, zero rate |
| Deployment | exists/closed, ACT-only deposit, invalid price, invalid group state |
| Market | too many bids, bid exists/price over order/invalid price, resource/attribute mismatch, order/bid/lease state, reclamation not started/not elapsed/invalid |
| BME/oracle | stale/unhealthy/zero price, circuit breaker status, vault insufficient, below minimum/precision |

Solana freezes Anchor error codes at IDL freeze; EVM uses custom-error selectors. Clients treat a closed-and-reaped Solana account-not-found as terminal closed where appropriate.

## Concept glossary

| Cosmos | Solana | EVM |
|---|---|---|
| Module/keeper | Program + owned PDAs | Contract + storage |
| Msg | Instruction | External function |
| KV store/index | Primary PDA; secondary indexer | Mapping; secondary indexer |
| Begin/EndBlocker | Permissionless crank | Keeper automation + permissionless call |
| Keeper hook | CPI or guarded reap | Cross-contract call/reap |
| Gov authority | Realms governance PDA/Squads | Governor + Timelock/Safe |
| Module params | Governed config PDA | `AkashConfig` |
| Module account | Program PDA/token account | Contract-held balance |
| Typed event | Event CPI | Solidity log |
| Genesis/export | Initialization + migration artifacts | Initializers + migration artifacts |
| Cosmovisor upgrade | Program loader upgrade | UUPS upgrade |
| Consensus version/store migration | Account version/lazy migrate | ERC-7201/reinitializer |
| Discovery | Config program registry | Config/address/ABI registry |

Use [03](./03-solana-architecture.md) and [04](./04-ethereum-architecture.md) for normative implementations, [05](./05-token-migration.md)–[06](./06-state-and-data-migration.md) for balance/state disposition, and [09](./09-testing-and-verification.md) to turn this table into parity coverage.
