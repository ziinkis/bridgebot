# Goal — Bridgebot Research Phase

## Primary Goal

Menentukan, berdasarkan data dan bukan asumsi, apakah **multi-venue inventory arbitrage** layak secara ekonomi dan bagaimana sistem paling aman, capital-efficient, dan reusable lintas chain harus dibangun.

**ALPH adalah first research profile / first implementation target, bukan identitas permanen core engine.**

Sistem target harus mampu membandingkan dan pada akhirnya mengeksekusi opportunity antara venue berbeda tanpa mengharuskan asset berpindah terlebih dahulu pada hot path, serta dapat dipakai pada economic asset dan chain lain dengan menambahkan registry/config/adapter — bukan rewrite core arbitrage engine.

## Initial Research Profile

### Chains yang sedang diteliti pertama

- Alephium
- Ethereum
- BNB Chain

### Venues yang sedang diteliti pertama

- Elexium
- AYIN
- Nightshade jika executable liquidity relevan
- Uniswap
- PancakeSwap
- venue tambahan hanya setelah terverifikasi

### Transfer / settlement profile pertama

- Alephium Bridge
- native chain transfer bila relevan
- future CEX deposit/withdrawal

Initial profile di atas adalah **research scope**, bukan hardcoded production universe.

---

# Design Law — No Chain-Specific Hardcoding in Core

Core engine tidak boleh memiliki compile-time assumption bahwa:

```text
chain = Alephium / Ethereum / BSC
asset = ALPH
venue = Elexium / Uniswap / PancakeSwap
bridge = Alephium Bridge
quote = USDT
```

Data berikut tidak boleh ditanam sebagai business-logic constants di core:

```text
chain id
network id
RPC / websocket URL
explorer URL
native gas asset
block/finality settings
contract address
token id / token contract
decimals
router address
factory address
pool address
pool list
fee tier
market symbol
bridge contract
transfer route
minimum trade size
maximum trade size
capital amount
allocation percentage
slippage percentage
minimum profit bps
gas budget
inventory threshold
rebalance threshold
quote max age
```

Semua harus berasal dari salah satu sumber runtime berikut:

```text
1. validated configuration
2. protocol/chain registry
3. on-chain discovery
4. authoritative venue API
5. measured runtime state
6. operator policy/config
```

Protocol-specific **behavior/logic** tetap boleh hidup dalam adapter, karena mekanisme Uniswap V3, Alephium contracts, CEX orderbooks, dll memang berbeda. Tetapi adapter tidak boleh membuat core arbitrage economics bergantung pada satu chain atau satu protocol.

---

# Generic Core Model

Core harus bekerja dengan abstraksi:

```text
Chain
Asset
SettlementAsset
EconomicAsset
Venue
Market
Quote
InventoryLocation
TransferProvider
ExecutionAdapter
RiskPolicy
```

Bukan:

```text
AlephiumEngine
AlphEngine
UniswapEngine
```

Target extensibility:

```text
NEW CHAIN
    ↓
register chain metadata
add chain adapter if new execution semantics are needed
register assets
register/discover venues
register transfer providers
    ↓
CORE ARBITRAGE ENGINE UNCHANGED
```

Contoh future profile:

```text
EconomicAsset = TOKEN_X
Chains = ChainA + ChainB + ChainC
Venues = DEX + CEX
Transfers = bridge + native transfer + CEX withdrawal
```

tanpa mengubah opportunity evaluator, sizing model, inventory economics, atau rebalance optimizer.

---

# Runtime Registry Model

Runtime harus membangun universe dari registry/state, misalnya:

```text
ChainRegistry
AssetRegistry
VenueRegistry
MarketRegistry
TransferRegistry
PolicyRegistry
```

Setiap registry record harus mempunyai:

```text
source
verification_state
version / updated_at
runtime health
```

Dynamic bukan berarti percaya data apa pun. Discovery/config baru harus melewati validation/allowlist/policy sebelum execution-enabled.

---

# Capital Model — Runtime Driven

Tidak ada fixed capital constraint seperti `1,000 ALPH-equivalent` dalam core design.

Angka nominal hanya boleh menjadi research/test scenario.

Bot harus dapat beroperasi dengan nominal apa pun yang benar-benar dialokasikan tanpa perubahan core logic.

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

---

# Dynamic Sizing Requirement

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

---

# What Must Be Optimized

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

---

# Core Hypotheses to Validate

1. Prefunded inventory lebih tepat daripada transfer-per-trade untuk non-atomic cross-location arbitrage.
2. Natural reverse arbitrage dan flow netting dapat mengurangi kebutuhan transfer secara material.
3. Rebalancing harus memilih asset/path termurah, bukan selalu memindahkan base asset.
4. Executable liquidity, bukan displayed spot spread, kemungkinan menjadi constraint utama untuk banyak capital regimes.
5. Dynamic sizing lebih profitable dan lebih aman daripada fixed trade size.
6. Dynamic slippage berdasarkan profit budget lebih aman daripada fixed tolerance.
7. Inventory depletion time + transfer latency distribution lebih tepat sebagai rebalance trigger daripada fixed balance percentage.
8. Venue abstraction harus cukup umum untuk DEX dan CEX.
9. Core engine harus capital-agnostic.
10. Core engine harus chain-agnostic dan asset-agnostic.
11. Menambah chain/asset/venue baru tidak boleh membutuhkan perubahan pada core opportunity economics.
12. Runtime discovery/configuration harus tetap dibatasi validation dan safety policy.

---

# Non-goals During Research Phase

Belum menjadi tujuan:

- real-money autonomous execution;
- private-key/signing architecture final;
- production deployment;
- aggressive latency optimization;
- mengasumsikan ALPH sebagai satu-satunya asset permanen;
- memilih satu chain sebagai chain permanen bot;
- membuat klaim ROI tanpa live dataset;
- memilih satu nominal modal sebagai nominal default bot.

---

# Research Success Criteria

Research cukup untuk masuk architecture/implementation phase jika kita dapat menjawab dengan evidence:

1. Venue mana yang benar-benar memiliki executable liquidity untuk research profile ALPH?
2. Berapa exact/actual fee per relevant pool/route?
3. Bagaimana output berubah terhadap size dari sangat kecil hingga mendekati usable inventory/liquidity capacity?
4. Berapa gas aktual per route dan percentile-nya?
5. Berapa quote drift p50/p90/p95/p99?
6. Berapa size yang memaksimalkan absolute economic net profit per route untuk capital state tertentu?
7. Berapa expected failed-leg/unwind loss?
8. Berapa transfer cost aktual per direction dan amount bucket?
9. Berapa settlement latency p50/p95/p99?
10. Kapan WAIT, NET, REVERSE TRADE, LOCAL SWAP, atau TRANSFER paling murah?
11. Berapa minimum inventory/trade-capacity yang aman sebagai fungsi trade-size distribution dan transfer latency?
12. Pada capital scenario kecil, sedang, dan besar, kapan strategi viable dan kapan `NO_TRADE`?
13. Dapatkah model yang sama direpresentasikan untuk chain/asset kedua tanpa mengubah core economics?
14. Apakah semua chain-specific data dapat dipindahkan ke registry/config/adapter boundary?

---

# Final Research Deliverable

```text
VERIFIED MARKET MAP (ALPH PROFILE)
        ↓
GENERIC ASSET / CHAIN / VENUE MODEL
        ↓
EXECUTABLE ECONOMICS MODEL
        ↓
CAPITAL-AGNOSTIC INVENTORY + REBALANCE MODEL
        ↓
CHAIN-AGNOSTIC ADAPTER / REGISTRY BOUNDARY
        ↓
EXECUTION RISK MODEL
        ↓
ECONOMIC VIABILITY
        ↓
ARCHITECTURE
```
