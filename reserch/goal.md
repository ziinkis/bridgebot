# Goal — Bridgebot Research Phase

## Primary Goal

Menentukan, berdasarkan data dan bukan asumsi, apakah **ALPH multi-venue inventory arbitrage** layak secara ekonomi dan bagaimana sistem paling aman serta capital-efficient harus dibangun.

Sistem target harus mampu membandingkan dan pada akhirnya mengeksekusi opportunity antara venue berbeda tanpa mengharuskan asset berpindah terlebih dahulu pada hot path.

## Initial Scope

### Chains

- Alephium
- Ethereum
- BNB Chain

### DEX venues

- Elexium
- AYIN
- Nightshade jika executable liquidity relevan
- Uniswap
- PancakeSwap
- venue tambahan hanya setelah terverifikasi

### Transfer / settlement

- Alephium Bridge
- native chain transfer bila relevan
- future CEX deposit/withdrawal

### Future extension

Core research dan architecture harus memungkinkan penambahan:

- CEX
- venue baru
- route baru
- quote asset baru

without rewriting the core arbitrage model.

## Capital Constraint

Starting research constraint:

```text
MAX CAPITAL PER ACTIVE VENUE = 1,000 ALPH-equivalent
```

Initial 50/50 base/quote split hanyalah **ASSUMPTION untuk eksperimen**, bukan keputusan final.

## What must be optimized

Sistem tidak boleh hanya memaksimalkan displayed spread.

Target objective:

```text
maximize ECONOMIC_NET_PNL
```

setelah memperhitungkan:

- executable buy/sell output
- LP/trading fees
- deterministic price impact
- gas
- adverse quote drift/slippage
- MEV/adverse selection
- failed-leg probability dan unwind loss
- inventory opportunity cost
- rebalance liability
- bridge/transfer cost
- bridge/transfer latency
- capital in transit

## Core hypotheses to validate

1. Prefunded inventory lebih tepat daripada bridge-per-trade untuk hot path cross-chain arbitrage.
2. Natural reverse arbitrage dan flow netting dapat mengurangi kebutuhan bridge secara material.
3. Rebalancing harus memilih asset/path termurah, bukan selalu memindahkan ALPH.
4. Executable liquidity, bukan displayed spot spread, kemungkinan menjadi constraint utama untuk modal kecil.
5. Dynamic sizing lebih profitable daripada fixed trade size.
6. Dynamic slippage berdasarkan profit budget lebih aman daripada fixed 0.5–1% tolerance.
7. Inventory depletion time + transfer p95 latency lebih tepat sebagai bridge trigger daripada fixed balance percentage.
8. Venue abstraction harus cukup umum untuk DEX dan future CEX.

## Non-goals during research phase

Belum menjadi tujuan:

- real-money autonomous execution;
- private-key/signing architecture final;
- production deployment;
- aggressive latency optimization;
- multi-strategy expansion di luar ALPH sebelum economics ALPH terukur;
- membuat klaim ROI tanpa live dataset.

## Research success criteria

Research dianggap cukup untuk masuk architecture/implementation phase jika kita dapat menjawab dengan evidence:

1. Venue mana yang benar-benar memiliki executable ALPH liquidity?
2. Berapa exact/actual fee per relevant pool/route?
3. Berapa net output untuk size 5/10/20/40/75/100/125/150 ALPH?
4. Berapa gas aktual per route dan percentile-nya?
5. Berapa quote drift p50/p90/p95/p99?
6. Berapa size yang memaksimalkan absolute net profit per route?
7. Berapa expected failed-leg/unwind loss?
8. Berapa bridge cost aktual per direction dan amount bucket?
9. Berapa source-confirm → VAA → redeem latency p50/p95/p99?
10. Kapan WAIT, NET, REVERSE TRADE, LOCAL SWAP, atau BRIDGE paling murah?
11. Berapa minimum inventory/trade-capacity yang aman per venue?
12. Apakah setelah semua cost dan risk reserve masih ada opportunity yang cukup sering dan cukup besar untuk layak dibangun?

## Final research deliverable

Research phase harus menghasilkan:

```text
VERIFIED MARKET MAP
        ↓
EXECUTABLE ECONOMICS MODEL
        ↓
INVENTORY + REBALANCE MODEL
        ↓
EXECUTION RISK MODEL
        ↓
ECONOMIC VIABILITY DECISION
        ↓
ARCHITECTURE
```
