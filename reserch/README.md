# Bridgebot Research Workspace

Folder `reserch/` adalah workspace research-first untuk proyek **Bridgebot**. Belum ada keputusan implementasi yang boleh dianggap final sebelum didukung bukti yang cukup.

## Tujuan

Membangun dasar faktual untuk **ALPH multi-venue inventory arbitrage engine** yang pada tahap awal mencakup:

- Alephium native ALPH
- Alephium DEX: Elexium, AYIN, Nightshade, dan venue lain yang terverifikasi
- Ethereum / Uniswap
- BNB Chain / PancakeSwap
- Alephium Bridge
- Ekstensi CEX di kemudian hari tanpa rewrite core architecture

## Capital model

**Tidak ada modal nominal yang di-hardcode ke desain bot.**

Angka seperti `100`, `500`, `1,000`, atau `100,000 ALPH-equivalent` hanya boleh digunakan sebagai **research scenario / test bucket**, bukan sebagai asumsi runtime.

Bot harus bekerja dengan modal yang benar-benar dialokasikan saat runtime:

```text
operator/wallet provides capital
        ↓
read actual balances
        ↓
subtract locked/reserved/in-transit/gas safety reserve
        ↓
derive usable allocated capital
        ↓
dynamically size opportunities
```

Contoh:

```text
allocated capital today = 100 ALPH-equivalent
allocated capital later = 1,000
allocated capital another deployment = 100,000
```

Core arbitrage logic harus tetap sama.

`wallet_balance` tidak otomatis berarti seluruh balance boleh digunakan. Pada wallet/account bersama, operator harus dapat menentukan allocation ceiling. Pada dedicated execution wallet, usable capital dapat diturunkan dari actual balance setelah reserve dan safety constraints.

## Research discipline

Setiap klaim harus diberi salah satu status berikut:

- **VERIFIED FACT** — berasal dari contract, protocol docs, chain data, atau sumber primer yang dapat diverifikasi.
- **MEASURED DATA** — hasil observasi/quote/gas/latency yang benar-benar diukur.
- **ASSUMPTION** — parameter sementara untuk simulasi; tidak boleh dianggap fakta.
- **UNKNOWN** — belum cukup bukti.
- **DECISION** — keputusan desain yang sudah memiliki evidence dan dicatat di `decisions/architecture_decisions.md`.

## Workflow

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

Dilarang membalik workflow menjadi `assume → code → baru ukur` untuk komponen yang menyentuh real-money execution.

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

## Core principle

Bot ini tidak diperlakukan sebagai "bridge-per-trade bot". Baseline hypothesis yang sedang diuji adalah:

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

Bridge, CEX withdrawal, CEX deposit, dan native transfer adalah **settlement/rebalancing edges**, bukan otomatis bagian hot path.

## Capital-agnostic sizing rule

Trade size tidak berasal dari angka modal tetap.

Secara konseptual:

```text
q_max = min(
    buy-side usable capacity,
    sell-side usable capacity,
    executable-liquidity safe size,
    per-trade risk limit,
    venue/transfer constraints
)

q* = argmax ECONOMIC_NET(q),  0 < q <= q_max
```

Jika modal terlalu kecil untuk mengalahkan fee/gas/risk, keputusan yang benar adalah **NO TRADE**, bukan memaksa size minimum.

## Data rule

Semua angka yang berubah terhadap waktu — liquidity, executable output, gas, fee konfigurasi, latency, withdrawal status, bridge health — harus mempunyai timestamp dan source/state reference. Jangan mengubah snapshot menjadi konstanta permanen.

## Coding gate

Pembuatan `src/` untuk execution engine ditunda sampai minimum berikut tersedia:

1. market map terverifikasi;
2. asset identity/provenance map;
3. executable quote model across dynamic size ranges;
4. fee + gas model;
5. bridge/transfer cost dan latency distribution;
6. capital-agnostic inventory model;
7. failed-leg/unwind model;
8. minimal economic viability test across multiple capital scenarios.
