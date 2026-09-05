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

## Capital Model — Runtime Driven

Tidak ada fixed capital constraint seperti `1,000 ALPH-equivalent` dalam core design.

Angka nominal hanya boleh menjadi research/test scenario.

Bot harus dapat beroperasi dengan nominal yang berubah, misalnya:

```text
100 ALPH-equivalent
500 ALPH-equivalent
1,000 ALPH-equivalent
100,000 ALPH-equivalent
atau nominal lain yang benar-benar dialokasikan
```

tanpa mengubah core logic.

Runtime source of truth:

```text
ACTUAL BALANCE
        ↓
minus locked/reserved/pending/in-transit
        ↓
minus native-gas / emergency reserve
        ↓
apply operator allocation ceiling if configured
        ↓
USABLE ALLOCATED CAPITAL
```

Important distinction:

```text
wallet balance != automatically spendable capital
```

Jika wallet/account dedicated hanya untuk bot, available balance dapat menjadi basis allocation setelah reserve. Jika wallet/account juga digunakan untuk hal lain, operator harus dapat memberi ceiling eksplisit agar bot tidak memakai dana di luar alokasi.

## Dynamic Sizing Requirement

Bot tidak boleh mengasumsikan nominal trade tetap ataupun persentase modal tetap.

Untuk setiap opportunity:

```text
q_max = min(
    usable buy-side inventory,
    usable sell-side inventory,
    executable-liquidity safe size,
    per-trade risk limit,
    venue constraints
)

q* = argmax ECONOMIC_NET(q)
subject to 0 < q <= q_max
```

Jika modal yang tersedia terlalu kecil sehingga fee, gas, impact, atau risk membuat `ECONOMIC_NET <= 0`, keputusan yang benar adalah `NO_TRADE`.

Jika modal sangat besar, bot tetap tidak boleh memaksa seluruh modal masuk trade; executable liquidity dan risk limit tetap menjadi pembatas.

## What must be optimized

Sistem tidak boleh hanya memaksimalkan displayed spread atau percentage return.

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
- capital utilization

## Core hypotheses to validate

1. Prefunded inventory lebih tepat daripada bridge-per-trade untuk hot path cross-chain arbitrage.
2. Natural reverse arbitrage dan flow netting dapat mengurangi kebutuhan bridge secara material.
3. Rebalancing harus memilih asset/path termurah, bukan selalu memindahkan ALPH.
4. Executable liquidity, bukan displayed spot spread, kemungkinan menjadi constraint utama untuk banyak capital regimes.
5. Dynamic sizing lebih profitable dan lebih aman daripada fixed trade size.
6. Dynamic slippage berdasarkan profit budget lebih aman daripada fixed 0.5–1% tolerance.
7. Inventory depletion time + transfer p95 latency lebih tepat sebagai bridge trigger daripada fixed balance percentage.
8. Venue abstraction harus cukup umum untuk DEX dan future CEX.
9. Core engine harus capital-agnostic: nominal allocation adalah runtime input/state, bukan compile-time/design constant.

## Non-goals during research phase

Belum menjadi tujuan:

- real-money autonomous execution;
- private-key/signing architecture final;
- production deployment;
- aggressive latency optimization;
- multi-strategy expansion di luar ALPH sebelum economics ALPH terukur;
- membuat klaim ROI tanpa live dataset;
- memilih satu nominal modal sebagai nominal "default bot".

## Research success criteria

Research dianggap cukup untuk masuk architecture/implementation phase jika kita dapat menjawab dengan evidence:

1. Venue mana yang benar-benar memiliki executable ALPH liquidity?
2. Berapa exact/actual fee per relevant pool/route?
3. Bagaimana output berubah terhadap size dari sangat kecil hingga mendekati usable inventory/liquidity capacity?
4. Berapa gas aktual per route dan percentile-nya?
5. Berapa quote drift p50/p90/p95/p99?
6. Berapa size yang memaksimalkan absolute economic net profit per route untuk capital state tertentu?
7. Berapa expected failed-leg/unwind loss?
8. Berapa bridge cost aktual per direction dan amount bucket?
9. Berapa source-confirm → VAA → redeem latency p50/p95/p99?
10. Kapan WAIT, NET, REVERSE TRADE, LOCAL SWAP, atau BRIDGE paling murah?
11. Berapa minimum inventory/trade-capacity yang aman per venue sebagai fungsi dari trade-size distribution dan transfer latency?
12. Pada capital scenario kecil, sedang, dan besar, kapan strategi tetap viable dan kapan harus `NO_TRADE`?
13. Apakah setelah semua cost dan risk reserve masih ada opportunity yang cukup sering dan cukup besar untuk layak dibangun?

## Final research deliverable

Research phase harus menghasilkan:

```text
VERIFIED MARKET MAP
        ↓
EXECUTABLE ECONOMICS MODEL
        ↓
CAPITAL-AGNOSTIC INVENTORY + REBALANCE MODEL
        ↓
EXECUTION RISK MODEL
        ↓
ECONOMIC VIABILITY ACROSS CAPITAL REGIMES
        ↓
ARCHITECTURE
```
