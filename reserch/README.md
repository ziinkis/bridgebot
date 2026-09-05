# Bridgebot Research Workspace

Folder `reserch/` adalah workspace research-first untuk proyek **Bridgebot**. Belum ada keputusan implementasi yang boleh dianggap final sebelum didukung bukti yang cukup.

## Project Direction

Target akhir bukan bot yang hardcoded untuk ALPH/Alephium.

Targetnya adalah:

```text
GENERIC MULTI-CHAIN / MULTI-VENUE
INVENTORY ARBITRAGE ENGINE
```

Dengan:

```text
ALPH + Alephium/Ethereum/BSC
```

sebagai **first research / validation profile**.

Core engine harus dapat dipakai untuk chain, asset, DEX, CEX, dan transfer network lain dengan menambah atau mengganti registry/config/adapter — bukan rewrite core economics.

---

## First Research Profile

Saat ini evidence difokuskan pada:

- Alephium native ALPH
- Elexium, AYIN, Nightshade dan venue Alephium lain yang terverifikasi
- Ethereum / Uniswap
- BNB Chain / PancakeSwap
- Alephium Bridge
- future CEX extension

Ini adalah research scope pertama, **bukan hardcoded runtime universe**.

---

## Design Laws

### 1. Research first

```text
RESEARCH
   ↓
MEASURE
   ↓
VERIFY
   ↓
COMPARE
   ↓
DECIDE
   ↓
ARCHITECTURE
   ↓
IMPLEMENT
```

### 2. No chain-specific hardcoding in core

Core business logic tidak boleh hardcode:

```text
chain/network ids
RPC/websocket URLs
asset/token ids
decimals
contract addresses
router/factory/pool addresses
pool lists
fee tiers
bridge contracts
market symbols
capital amount
trade size
slippage/profit thresholds
gas budgets
inventory/rebalance bands
```

Data runtime berasal dari validated registry/config, discovery, authoritative venue state, measured runtime state, dan operator policy.

### 3. Protocol behavior belongs in adapters

`No hardcoded data` tidak berarti setiap protocol mempunyai implementation sama.

Behavior yang memang adapter-specific misalnya:

```text
Uniswap V3 concentrated-liquidity math
Alephium UTXO/contract semantics
CEX orderbook sequencing
bridge/VAA lifecycle
```

Core detector/evaluator/sizer/inventory/rebalancer tetap generic.

### 4. Dynamic must fail closed

```text
DISCOVER
→ VERIFY IDENTITY
→ VERIFY PROTOCOL
→ VALIDATE CONFIG
→ VERIFY QUOTE
→ SIMULATE/PREFLIGHT
→ EXECUTION ALLOWED
```

Dynamic discovery tidak otomatis trusted.

### 5. Capital is runtime state

Tidak ada modal nominal permanent.

```text
actual balances
- locked/reserved/pending/in-transit
- gas/emergency reserve
- optional operator ceiling
= usable allocated capital
```

`wallet_balance` tidak otomatis berarti seluruh balance boleh digunakan.

---

## Generic Core Concepts

Architecture research harus mengarah ke:

```text
Chain
EconomicAsset
SettlementAsset
Venue
Market
ExecutableQuote
InventoryLocation
TransferProvider
ExecutionAdapter
RiskPolicy
```

Runtime registries:

```text
ChainRegistry
AssetRegistry
VenueRegistry
MarketRegistry
TransferRegistry
PolicyRegistry
```

Core tidak boleh mempunyai permanent `if chain == ALEPHIUM` atau `if token == ALPH` dalam opportunity economics.

---

## Capital-Agnostic Sizing

```text
q_max = min(
    buy-side usable capacity,
    sell-side usable capacity,
    executable-liquidity safe size,
    per-trade risk limit,
    venue constraints
)

q* = argmax ECONOMIC_NET(q), 0 < q <= q_max
```

Jika modal terlalu kecil untuk mengalahkan fee/gas/risk, keputusan benar adalah `NO_TRADE`.

Jika modal sangat besar, liquidity/risk dapat membatasi size jauh sebelum seluruh modal dipakai.

---

## Research Discipline

Setiap klaim:

- **VERIFIED FACT** — source primer/chain/protocol evidence.
- **MEASURED DATA** — hasil quote/gas/latency/state yang diukur.
- **ASSUMPTION** — parameter sementara simulasi.
- **UNKNOWN** — evidence belum cukup.
- **DECISION** — constraint desain di `decisions/architecture_decisions.md`.

Semua data yang berubah terhadap waktu harus mempunyai timestamp dan source/state reference.

---

## Struktur

```text
reserch/
├── README.md
├── goal.md
├── research/
│   ├── research_v1.md
│   ├── research_index.md
│   ├── 01_market_map.md
│   ├── 02_assets.md
│   ├── 03_venues.md
│   ├── 04_dex_fees.md
│   ├── 05_liquidity.md
│   ├── 06_price_impact.md
│   ├── 07_slippage.md
│   ├── 08_gas.md
│   ├── 09_bridge.md
│   ├── 10_inventory.md
│   ├── 11_rebalancing.md
│   ├── 12_execution_risk.md
│   ├── 13_cex_extension.md
│   └── open_questions.md
├── data/
│   ├── pools/
│   ├── quotes/
│   ├── gas/
│   ├── bridge/
│   └── experiments/
└── decisions/
    └── architecture_decisions.md
```

---

## Current Economic Thesis

```text
PREFUNDED INVENTORY
        +
EXECUTABLE QUOTING
        +
DYNAMIC SIZE OPTIMIZATION
        +
INVENTORY-AWARE ARBITRAGE
        +
NATURAL FLOW NETTING
        +
MIN-COST REBALANCING
```

Bridge, native transfer, CEX withdrawal/deposit, dan transfer provider lain adalah settlement/rebalancing edges, bukan otomatis bagian hot path.

---

## Coding Gate

Pembuatan `src/` execution engine ditunda sampai minimum tersedia:

1. first-profile market map terverifikasi;
2. generic chain/asset/venue identity model;
3. executable quote model across dynamic size ranges;
4. fee + gas model;
5. transfer cost + latency distributions;
6. capital-agnostic inventory model;
7. failed-leg/unwind model;
8. chain-agnostic adapter/registry boundary;
9. economic viability test;
10. proof bahwa second chain/asset profile dapat direpresentasikan tanpa rewrite core economics.
