# Architecture Decisions

Dokumen ini hanya berisi keputusan yang sudah cukup kuat untuk dijadikan constraint desain. Parameter yang masih eksperimen tetap berada di research files.

---

## ADR-001 — Research-first before execution implementation

**Status:** ACCEPTED

### Decision

Jangan membuat real-money execution architecture sebelum market, fee, liquidity, gas, bridge, inventory, dan failed-leg economics memiliki evidence minimum.

### Rationale

Cross-chain arbitrage mudah terlihat profitable pada displayed spread tetapi menjadi negatif setelah executable depth, fee, gas, quote drift, failed-leg risk, dan rebalance cost dihitung.

---

## ADR-002 — Core abstraction is Venue, not DEX

**Status:** ACCEPTED

### Decision

Core research/architecture harus memperlakukan tempat execution sebagai `Venue`.

```text
Venue
├── DEX
└── future CEX
```

### Rationale

CEX harus dapat ditambahkan tanpa rewrite opportunity economics.

---

## ADR-003 — Core transfer abstraction is broader than Bridge

**Status:** ACCEPTED

### Decision

Bridge adalah salah satu transfer/rebalance edge.

```text
Transfer
├── bridge
├── native transfer
├── future CEX deposit
└── future CEX withdrawal
```

### Rationale

Rebalancing harus memilih path termurah, bukan selalu bridge ALPH.

---

## ADR-004 — Separate EconomicAsset from SettlementAsset

**Status:** ACCEPTED

### Decision

Do not use ticker symbol as authoritative identity.

Example:

```text
EconomicAsset = ALPH
SettlementAssets = native ALPH, Ethereum wALPH, BSC wALPH, future CEX balances
```

### Rationale

Location, contract, bridge provenance, decimals, and settlement behavior differ.

---

## ADR-005 — Prefunded inventory is baseline hot-path model

**Status:** ACCEPTED AS RESEARCH BASELINE

### Decision

Normal opportunity evaluation assumes buy and sell inventories already exist at their respective venues.

Bridge/withdrawal is not a mandatory step between buy and sell.

### Rationale

Cross-chain transfer latency is incompatible with a deterministic arbitrage lock unless settlement is prefunded.

### Caveat

Economic superiority still must be validated with measured capital utilization and rebalance cost.

---

## ADR-006 — Use executable quotes, not displayed prices

**Status:** ACCEPTED

### Decision

Opportunity economics use amount-dependent executable quote/output for both legs.

### Rationale

Fee tier, concentrated liquidity, route, and price impact vary by size.

---

## ADR-007 — Dynamic sizing

**Status:** ACCEPTED

### Decision

Do not define one permanent trade size.

Search for size that maximizes economic net profit under inventory/risk constraints.

### Rationale

Absolute profit often peaks before maximum possible trade size because price impact increases nonlinearly.

---

## ADR-008 — Rebalance only net inventory liability

**Status:** ACCEPTED

### Decision

Natural reverse flow and netting happen before bridge/transfer decisions.

### Rationale

Gross directional flows can be much larger than final net settlement requirement.

---

## ADR-009 — Inventory is base + quote + gas + state

**Status:** ACCEPTED

### Decision

Do not model only ALPH inventory.

Track base, quote, native gas, reserved, pending, in-transit, and settled balances independently.

### Rationale

Bridging ALPH alone can restore base balance while quote-side trade capacity remains depleted.

---

## ADR-010 — In-transit funds are not available inventory

**Status:** ACCEPTED

### Decision

Incoming bridge/transfer balance becomes usable only after destination settlement/reconciliation.

---

## ADR-011 — Bridge decision is economic + depletion-aware

**Status:** ACCEPTED AS MODEL DIRECTION

### Decision

Bridge trigger should consider:

```text
remaining trade capacity
time-to-inventory-floor
transfer p95 latency
pending flow
netting probability
bridge health
transfer cost
lost opportunity cost
```

not only a fixed percentage threshold.

### Caveat

Exact thresholds remain research outputs.

---

## ADR-012 — Slippage must preserve profit budget

**Status:** ACCEPTED

### Decision

Do not use a universal 0.5–1% slippage tolerance.

Tolerance is bounded by remaining profit after required final margin and risk reserve, then calibrated from measured adverse quote drift.

---

## ADR-013 — Failed-leg recovery is precomputed

**Status:** ACCEPTED

### Decision

Every non-atomic cross-venue candidate must have an unwind/hedge plan and bounded maximum loss before execution is considered production-ready.

---

## ADR-014 — Capital is runtime state, not a design constant

**Status:** ACCEPTED

### Decision

Core bot logic must not assume a permanent capital amount such as `1,000 ALPH-equivalent`.

Nominal values such as:

```text
100
500
1,000
100,000 ALPH-equivalent
```

may be used only as research/simulation scenarios.

At runtime, capital is derived from actual inventory state:

```text
actual balance
- locked
- reserved
- pending/in-transit exclusions
- gas reserve
- emergency reserve
= available balance
```

Then apply an optional operator-defined allocation ceiling:

```text
usable allocated capital
=
min(available balance, allocation ceiling)
```

when a ceiling is configured.

A dedicated execution wallet may allow the bot to derive usable allocation from its balance after reserves. A shared wallet/account must support an explicit ceiling so unrelated funds are never treated as bot capital.

Trade sizing must therefore be capital-agnostic:

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

### Rationale

The same engine should work whether the operator provides very small, medium, or very large inventory. At small capital, fixed gas/fees may make the correct action `NO_TRADE`. At large capital, executable liquidity and risk may bind long before the full balance can be used.

### Supersedes

Any earlier research wording that describes `1,000 ALPH-equivalent per venue` as a capital constraint is superseded by this ADR. Such values are examples/scenario buckets only.

---

# Explicitly NOT decided yet

The following remain open and must not be presented as final architecture constants:

```text
any fixed nominal capital per venue
50/50 venue allocation
75–100 ALPH normal trade size
exact inventory warning/hard bands
minimum net bps
bridge batch size
bridge cost budget
quote max age
gas reserve amount
p95/p99 slippage buffer
specific preferred Alephium DEX
specific preferred CEX
```

These belong to measured research, not permanent decisions.
