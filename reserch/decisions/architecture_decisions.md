# Architecture Decisions

Dokumen ini hanya berisi keputusan yang sudah cukup kuat untuk dijadikan constraint desain. Parameter yang masih eksperimen tetap berada di research files.

---

## ADR-001 — Research-first before execution implementation

**Status:** ACCEPTED

### Decision

Jangan membuat real-money execution architecture sebelum market, fee, liquidity, gas, transfer, inventory, dan failed-leg economics memiliki evidence minimum.

---

## ADR-002 — Core abstraction is Venue, not DEX

**Status:** ACCEPTED

### Decision

Core memperlakukan tempat execution sebagai `Venue`.

```text
Venue
├── DEX
└── CEX
```

Menambah CEX atau DEX baru tidak boleh mengubah opportunity economics.

---

## ADR-003 — Core transfer abstraction is broader than Bridge

**Status:** ACCEPTED

```text
Transfer
├── bridge
├── native transfer
├── CEX deposit
└── CEX withdrawal
```

Rebalancing memilih transfer/path berdasarkan economics, capacity, latency, health, dan risk.

---

## ADR-004 — Separate EconomicAsset from SettlementAsset

**Status:** ACCEPTED

Ticker bukan authoritative identity.

```text
EconomicAsset
    abstract economic exposure

SettlementAsset
    exact representation at a chain/location
```

ALPH/wALPH adalah first-profile example, bukan permanent core type.

---

## ADR-005 — Prefunded inventory is baseline hot-path model

**Status:** ACCEPTED AS RESEARCH BASELINE

Normal opportunity evaluation mengasumsikan buy dan sell inventory sudah tersedia pada location masing-masing. Transfer bukan mandatory step di antara kedua legs.

Economic superiority tetap harus divalidasi dengan measured capital utilization dan rebalance cost.

---

## ADR-006 — Use executable quotes, not displayed prices

**Status:** ACCEPTED

Opportunity economics menggunakan amount-dependent executable quote/output.

---

## ADR-007 — Dynamic sizing

**Status:** ACCEPTED

Tidak ada permanent trade size. Size dicari untuk memaksimalkan economic net profit di bawah inventory, liquidity, venue, dan risk constraints.

---

## ADR-008 — Rebalance only net inventory liability

**Status:** ACCEPTED

Natural reverse flow dan netting dipertimbangkan sebelum transfer/rebalance.

---

## ADR-009 — Inventory is multi-asset + gas + state

**Status:** ACCEPTED

Track base/economic asset, quote/hedge assets, native gas, reserved, locked, pending, in-transit, redeemable, dan settled balances secara independen.

---

## ADR-010 — In-transit funds are not available inventory

**Status:** ACCEPTED

Incoming transfer baru menjadi usable setelah destination settlement dan reconciliation berhasil.

---

## ADR-011 — Rebalance decision is economic + depletion-aware

**Status:** ACCEPTED AS MODEL DIRECTION

Trigger mempertimbangkan:

```text
remaining trade capacity
time-to-inventory-floor
transfer latency distribution
pending flow
netting probability
transfer health
transfer cost
lost opportunity cost
```

bukan fixed percentage saja.

---

## ADR-012 — Slippage must preserve profit budget

**Status:** ACCEPTED

Tidak ada universal fixed slippage tolerance. Tolerance dibatasi remaining profit budget dan dikalibrasi dari measured adverse quote drift.

---

## ADR-013 — Failed-leg recovery is precomputed

**Status:** ACCEPTED

Setiap non-atomic cross-location candidate harus memiliki unwind/hedge plan dan bounded maximum loss sebelum production execution.

---

## ADR-014 — Capital is runtime state, not a design constant

**Status:** ACCEPTED

Core tidak boleh mengasumsikan permanent capital amount.

```text
actual balance
- locked
- reserved
- pending/in-transit exclusions
- gas reserve
- emergency reserve
= available balance
```

Optional operator allocation ceiling:

```text
usable allocated capital
=
min(available balance, allocation ceiling)
```

Trade sizing:

```text
q_max = min(
    buy-side usable inventory,
    sell-side usable inventory,
    liquidity-safe size,
    per-trade risk limit,
    venue constraints
)

q* = argmax ECONOMIC_NET(q), 0 < q <= q_max
```

Any earlier `1,000 ALPH-equivalent` wording is scenario-only and superseded as a design constraint.

---

## ADR-015 — Core engine is chain-agnostic and asset-agnostic

**Status:** ACCEPTED

### Decision

ALPH/Alephium adalah first research and implementation profile only.

Core engine tidak boleh memiliki domain model seperti:

```text
AlphTrade
AlephiumOpportunity
UniswapArbitrage
```

sebagai foundation economics.

Core model harus menggunakan generic identities/interfaces:

```text
ChainId
EconomicAssetId
SettlementAssetId
VenueId
MarketId
InventoryLocationId
TransferProviderId
Quote
ExecutionPlan
RiskPolicy
```

### Required extension behavior

Menambah chain baru harus mengikuti:

```text
register chain
register assets
add/reuse chain adapter
add/reuse venue adapters
register/discover markets
add/reuse transfer adapters
        ↓
core detector/evaluator/sizer/rebalancer unchanged
```

Jika penambahan chain baru memerlukan perubahan pada formula core arbitrage economics, architecture dianggap gagal memenuhi ADR ini kecuali memang ada primitive ekonomi baru yang generalizable.

---

## ADR-016 — No chain/protocol data hardcoded in core business logic

**Status:** ACCEPTED

### Decision

Core tidak boleh hardcode:

```text
chain/network ids
RPC/websocket URLs
explorer URLs
native gas asset ids
contract/token ids
decimals
router/factory addresses
pool addresses / pool lists
fee tiers
bridge contracts
market symbols
transfer paths
capital amounts
size buckets
slippage limits
profit thresholds
gas budgets
inventory bands
rebalance thresholds
quote-age limits
```

Data tersebut berasal dari runtime config, validated registries, authoritative APIs, on-chain discovery, measured state, atau operator policy.

### Important distinction

`No hardcoded data` tidak berarti `no protocol-specific code`.

Protocol semantics memang berbeda. Contoh:

```text
Uniswap V3 tick/liquidity math
Alephium UTXO/contract execution
CEX orderbook sequencing
Wormhole-style VAA lifecycle
```

Logic seperti ini berada di adapters. Tetapi address, deployment metadata, active markets, fee state, dan policy numbers tidak boleh tersembunyi sebagai constants di core economics.

---

## ADR-017 — Dynamic discovery must be validated before execution

**Status:** ACCEPTED

### Decision

Dynamic tidak berarti otomatis percaya semua hasil discovery.

Lifecycle:

```text
DISCOVERED
→ IDENTITY_VERIFIED
→ PROTOCOL_VERIFIED
→ CONFIG_VALIDATED
→ QUOTE_VERIFIED
→ SIMULATION_VERIFIED
→ EXECUTION_ALLOWED
```

Unknown/spoof pools, arbitrary tokens, untrusted RPC/config, atau changed contracts tidak boleh menjadi execution-eligible hanya karena ditemukan secara dinamis.

### Rationale

Full dynamism tanpa validation memperbesar attack surface. Reusability harus tetap fail-closed.

---

## ADR-018 — Runtime registries are first-class architecture components

**Status:** ACCEPTED

### Decision

Architecture harus mempunyai conceptual registries:

```text
ChainRegistry
AssetRegistry
VenueRegistry
MarketRegistry
TransferRegistry
PolicyRegistry
```

Every record carries provenance/state such as:

```text
source
verification_state
version / updated_at
health
```

Core engine membaca universe dari registries, bukan dari chain-specific `match`/`if` trees.

---

# Explicitly NOT decided yet

Tetap belum final:

```text
specific chain adapters implementation language/details
specific registry file/database format
config hot-reload mechanism
plugin loading mechanism
50/50 allocation
normal trade size
inventory warning/hard bands
minimum net bps
transfer batch size
transfer cost budget
quote max age
gas reserve amount
p95/p99 slippage buffer
preferred DEX/CEX
```

Semua numeric operating parameters harus measured/policy-driven, bukan permanent business-logic constants.
