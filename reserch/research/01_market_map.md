# 01 — Market Map

**Research status:** `IN_PROGRESS / PARTIALLY_VERIFIED`  
**Snapshot date:** `2026-09-05`  
**Primary economic asset:** `ALPH`  
**Capital constraint under research:** `<= 1,000 ALPH-equivalent per active venue`

---

## 0. Purpose

Membangun **evidence-backed market graph** untuk seluruh lokasi yang secara nyata dapat memperdagangkan economic asset `ALPH`.

Dokumen ini tidak menganggap sebuah market layak hanya karena:

- muncul di UI,
- pernah diumumkan,
- memiliki contract/pool address,
- memiliki headline TVL,
- atau memiliki displayed price.

Market map harus menjawab:

```text
WHERE can ALPH be traded?
WHAT exact settlement asset is traded there?
WHICH pool / market / factory owns that liquidity?
WHAT fee tier applies?
HOW is the route discovered and quoted?
IS there executable liquidity for our actual size?
CAN inventory later be transferred/rebalanced?
```

Market discovery dan execution eligibility adalah dua hal berbeda.

---

# 1. Evidence states

Gunakan state berikut di seluruh research.

```text
IDENTITY_VERIFIED
    asset identity / provenance terverifikasi

MARKET_VERIFIED
    market/pool benar-benar ada dan pair identity terverifikasi

DISCOVERY_VERIFIED
    factory/registry dapat digunakan untuk menemukan markets secara deterministik

QUOTE_REQUIRED
    market ada, tetapi live executable quote belum diukur

WATCH
    market valid tetapi economics/liquidity belum cukup untuk execution allowlist

REJECT_LIQUIDITY
    market valid tetapi current liquidity tidak memadai

REJECT_FEE
    market valid tetapi fee membuatnya tidak masuk normal arbitrage universe

DISCOVERY_ONLY
    ditemukan tetapi belum bagian dari active execution scope

UNRESOLVED
    belum cukup evidence untuk attribution / execution
```

Tidak ada market yang boleh menjadi `EXECUTION_ELIGIBLE` hanya dari indexer snapshot. Final eligibility membutuhkan live quote + simulation/preflight.

---

# 2. Current official venue universe

Current Alephium documentation menyebut ALPH tersedia:

```text
Native Alephium
├── Elexium
├── AYIN
└── Nightshade

Ethereum
└── Uniswap — wrapped ALPH

BNB Chain
└── PancakeSwap — wrapped ALPH
```

Official documentation juga secara eksplisit memperingatkan bahwa beberapa pair mempunyai liquidity rendah.

**Primary source:**
- https://docs.alephium.org/

Ini adalah venue discovery universe, **bukan bukti bahwa semua venue tersebut saat ini executable**.

---

# 3. Economic asset vs settlement asset

Arbitrage engine harus membedakan:

```text
EconomicAsset
    ALPH

SettlementAsset
    exact token/coin at a specific location
```

Economic equivalence tidak berarti settlement equivalence.

## 3.1 Canonical ALPH settlement assets

| Location | Settlement asset | Identifier | Decimals | Status |
|---|---|---|---:|---|
| Alephium | native ALPH | `0000000000000000000000000000000000000000000000000000000000000000` | 18 | `IDENTITY_VERIFIED` |
| Ethereum | wALPH / Alephium (AlphBridge) | `0x590F820444fA3638e022776752c5eEF34E2F89A6` | 18 | `IDENTITY_VERIFIED` |
| BNB Chain | wALPH / Alephium (AlphBridge) | `0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8` | 18 | `IDENTITY_VERIFIED` |

Evidence:

- Alephium token list identifies native ALPH as the all-zero token id with 18 decimals.
- Etherscan identifies Ethereum wALPH as an ERC-20 proxy with 18 decimals.
- BscScan identifies BSC wALPH as a BEP-20 proxy with 18 decimals.
- Alephium's bridge verification article identifies the Ethereum/BSC wrapped token addresses as official bridge assets.

Sources:
- https://github.com/alephium/token-list/blob/master/tokens/mainnet.json
- https://etherscan.io/token/0x590f820444fa3638e022776752c5eef34e2f89a6
- https://bscscan.com/token/0x8683ba2f8b0f69b2105f26f488bade1d3ab4dec8
- https://alephium.org/news/post/verification-of-bridge-contracts-tokens-token-lists-76e5c237bf52/

---

# 4. Quote-asset provenance on Alephium

Do **not** normalize token identity only by ticker.

Current official Alephium token list distinguishes bridge origin.

| Symbol used by ecosystem | Token ID | Decimals | Provenance |
|---|---|---:|---|
| `USDTeth` | `556d9582463fe44fbd108aedc9f409f69086dc78d994b88ea6c9e65f8bf98e00` | 6 | Ethereum via Alephium Bridge |
| `USDTbsc` | `7ff5e72636f640eb2c28056df3b6879e4c86933505abebf566518ad396335700` | 18 | BSC via Alephium Bridge |
| `USDCeth` | `722954d9067c5a5ad532746a024f2a9d7a18ed9b90e27d0a3a504962160b5600` | 6 | Ethereum via Alephium Bridge |
| `USDCbsc` | `75e1e9f91468616a371fe416430819bf5386a3e6a258864c574271a404ec8900` | 18 | BSC via Alephium Bridge |
| `WETH` | `19246e8c2899bc258a1156e08466e3cdd3323da756d8a543c7fc911847b96f00` | 18 | Ethereum asset bridged to Alephium |
| `WBTC` | `383bc735a4de6722af80546ec9eeb3cff508f2f68e97da19489ce69f3e703200` | 8 | Ethereum asset bridged to Alephium |

Therefore:

```text
USDTeth != USDTbsc
```

for settlement/accounting purposes even if both are valued near USD 1.

**Primary source:**
- https://github.com/alephium/token-list/blob/master/tokens/mainnet.json

Important: Alephium's token-list validates metadata format; inclusion itself is not a security audit. Canonical/provenance checks still need protocol-specific evidence.

---

# 5. Bridge is a transfer edge, not a market

The Alephium Bridge must be represented separately from trading venues.

Canonical bridge contracts currently documented:

| Chain | Component | Address |
|---|---|---|
| Ethereum | TokenBridge | `0x579a3bDE631c3d8068CbFE3dc45B0F14EC18dD43` |
| Ethereum | BridgeImplementation | `0x0F843945075DF4EA9C8a21f0e0CcFD5eB073eEAb` |
| Ethereum | TokenImplementation | `0xdeb8c2C57c7de48D3Ad5a980be3DD23868262b6A` |
| Alephium | TokenBridge | `23Fj7xr1pxWfYLixz3aBC3u5dUJVpAjXArbpiYWxeGjQT` |
| BNB Chain | TokenBridge | `0x2971F580C34d3D584e0342741c6a622f69424dD8` |

Source:
- https://alephium.org/news/post/verification-of-bridge-contracts-tokens-token-lists-76e5c237bf52/

Current documented bridge timing baseline:

```text
ETH -> ALPH   ~15–20 minutes average
BSC -> ALPH   ~20 minutes
ALPH -> ETH   >=205 blocks AND >=28 minutes
ALPH -> BSC   >=205 blocks AND >=28 minutes
```

Source:
- https://docs.alephium.org/infrastructure/the-bridge/

Implication for market map:

```text
trade graph != transfer graph
```

Cross-venue arbitrage should be evaluated using prefunded inventory. Bridge paths are later rebalancing edges.

---

# 6. Native Alephium market discovery

The key design decision from this research is:

> Native Alephium pools must be enumerated from protocol factory/registry state, not maintained as a permanent manual list.

This allows new pools to appear without code changes and prevents stale UI assumptions.

## 6.1 Elexium

Current DefiLlama Alephium adapter identifies:

```text
Elexium Pair Factory
22oTtDJEMjNc9QAdmcZarnEzgkAooJp9gZy7RYBisniR5
```

The adapter calls the factory to obtain pair count and iterates pool ids dynamically.

Additional protocol characteristics from Elexium documentation:

- normal pools use a modified v2-style constant-product model;
- stable-pool support exists;
- the router is designed for multi-pool routing and documented as tested up to 8 hops.

Sources:
- https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/elexium/index.js
- https://docs.elexium.finance/

Status:

```text
Venue identity        VERIFIED
Factory discovery     DISCOVERY_VERIFIED
Individual ALPH pools DYNAMIC_ENUMERATION_REQUIRED
Live executable depth QUOTE_REQUIRED
```

Do not hardcode a single Elexium ALPH market.

---

## 6.2 AYIN Classic / V2-style pools

Current DefiLlama adapter explicitly identifies these classic AMM pools:

| Pair attribution in adapter | Pool address | Market-map relevance |
|---|---|---|
| ALPH / AYIN | `25ywM8iGxKpZWuGA5z6DXKGcZCXtPBmnbQyJEsjvjjWTy` | secondary quote route |
| ALPH / USDTeth | `2A5R8KZQ3rhKYrW7bAS4JTjY9FCFLJg6HjQpqSFZBqACX` | **primary USD candidate** |
| ALPH / USDCeth | `283R192Z8n6PhXSpSciyvCsLEiiEVFkSE6MbRBA4KSaAj` | USD candidate |
| ALPH / WETH | `yXMFxdoKcE86W9NAyajc8Z3T3k2f5FGiHqHtuA69DYT1` | ETH route candidate |
| ALPH / WBTC | `28XY326TxvSekaAwiWDLFg2QBRfacSga8dyNJCYGUYNbq` | BTC route candidate |
| ALPH / APAD | `vFpZ1DF93x1xGHoXM8rsDBFjpcoSsCi5ZEuA5NG5UJGX` | secondary |
| ALPH / CHENG | `25b5aNfdrNRjJ7ugPTkxThT51L1NSvf8igQyDHKZhweiK` | secondary |
| ALPH / ANSd | `uM4QJwHqFoTF2Pou8TqwhaDiHYLk4SHG65uaQG8r7KkT` | secondary |
| ALPH / ALPHAGA | `23cXw23ZjRqKc7i185ZoH8vh9KT4XTumVRWpVLUecgLMd` | secondary |
| AYIN / USDTeth | `21NEBCk8nj5JBKpS7eN8kX6xGJoLHNqTS3WBFnZ7q8L9m` | possible multi-route component |
| AYIN / USDCeth | `2961aauvprhETv6TXGQRc3zZY4FbLnqKon2a4wK6ABH9q` | possible multi-route component |
| USDTeth / USDCeth | `27C75V9K5o9CkkGTMDQZ3x2eP82xnacraEqTYXA35Xuw5` | stable conversion component |

Source:
- https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/ayin/index.js

The ALPH/USDTeth pool `2A5R8K...BqACX` is independently visible in Alephium Explorer swap transactions. Example March 2026 transactions show successful 100 ALPH swaps against this contract, confirming this is not merely a stale label in an adapter.

Explorer example:
- https://explorer.alephium.org/transactions/38de7cabfa8454e9edfbf49844616f557f0fd5ef2715b9a68ef9d786ea5ac1df

Important:

```text
historical reserve != current executable liquidity
```

The pool identity is verified; current depth still requires a live quote/state snapshot.

---

## 6.3 AYIN concentrated-liquidity universe

Current AYIN concentrated-liquidity factory identified by the live DefiLlama adapter:

```text
AYIN CL Factory
z73CeQLRpbaQ5gF7bJ1yoYqmmonwC9h9wEwLxGF9EjVy
```

Its child/sub-contract pools are dynamically enumerated through Alephium node APIs.

AYIN documentation identifies v3/concentrated fee tiers:

```text
0.01%
0.05%
0.30%
1.00%
```

AYIN Smart Router v1 considers v2 + concentrated pools and can split a trade across same-pair liquidity sources. Cross-pair hops are described as future/expanding functionality in the documentation, so the bot must verify current router behavior rather than assume every arbitrary multi-hop path exists.

Sources:
- https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/ayin/index.js
- https://docs.ayin.app/ayin/ayin-metaswap/what-is-smart-router
- https://docs.ayin.app/ayin/ayin-metadex/concentrated

Status:

```text
Classic pool identity     VERIFIED
CL factory discovery      DISCOVERY_VERIFIED
CL ALPH pool enumeration  PENDING DATA EXTRACTION
Best route per size       QUOTE_REQUIRED
```

---

## 6.4 Nightshade

Current DefiLlama adapter identifies Nightshade's main pair registry/contract as:

```text
24pPSeXvbmUjM6NZVYehEjLiCDDDeT94CR9rp1YJSuxKZ
```

The adapter reads pair count, token ids and balances dynamically from this contract.

Sources:
- https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/nightshade/index.js
- https://nightshade.finance/token/ALPH
- https://docs.nightshade.finance/

Status:

```text
Venue identity          VERIFIED
Registry discovery      DISCOVERY_VERIFIED
ALPH pair enumeration   PENDING DATA EXTRACTION
Executable depth        QUOTE_REQUIRED
```

Nightshade's UI confirms an ALPH trading surface, but individual current pool rows are not sufficiently exposed by the static page to treat UI scraping as source of truth. Use registry calls instead.

---

# 7. Native Alephium ecosystem-level liquidity context

DefiLlama snapshots around this research date show the native DEX ecosystem is small:

```text
Nightshade Finance  ~ $70k protocol TVL
AYIN                ~ $44k–46k protocol TVL
Elexium             ~ $40k–42k protocol TVL
```

Sources:
- https://defillama.com/chain/alephium
- https://defillama.com/protocol/ayin
- https://defillama.com/protocol/elexium

This does **not** mean an ALPH trade can use that whole TVL.

Protocol TVL includes:

- non-ALPH pairs,
- liquidity away from current CL price ranges,
- assets on the wrong side of a pair,
- and liquidity that may be economically unusable at our size.

Therefore the authoritative metric for the bot remains:

```text
ExecutableQuote(size, direction, current_state)
```

not protocol TVL.

---

# 8. Ethereum market map

Canonical wrapped asset:

```text
wALPH
0x590F820444fA3638e022776752c5eEF34E2F89A6
18 decimals
```

## 8.1 Uniswap V3 — ALPH/USDT

```text
Pool
0xa344855388C9f2760e998eb2207B58de6E7d0360

ALPH
0x590F820444fA3638e022776752c5eEF34E2F89A6

USDT
0xdAC17F958D2ee523a2206206994597C13D831ec7

Fee tier
1.00% = 100 bps
```

Recent indexer snapshot (not execution source of truth):

```text
Liquidity ~ $20.4k
24h volume ~ $57.5
Pool composition heavily skewed toward ALPH side
```

Status:

```text
MARKET_VERIFIED
QUOTE_REQUIRED
CANDIDATE_FOR_LIVE_QUOTE_LADDER
```

Source:
- https://www.geckoterminal.com/eth/pools/0xa344855388c9f2760e998eb2207b58de6e7d0360

Critical observation: headline liquidity is asymmetric. A pool can report ~$20k total while having only about ~$1k of the quote-side asset in the observed snapshot. Direction-specific executable depth matters more than total TVL.

---

## 8.2 Uniswap V3 — ALPH/WETH

```text
Pool
0x9628105808292699874f20d77d50a09bc26850c5

ALPH
0x590F820444fA3638e022776752c5eEF34E2F89A6

WETH
0xC02aaA39b223FE8D0A0e5C4F27eAD9083C756Cc2

Fee tier
1.00% = 100 bps
```

Recent indexer snapshot:

```text
Liquidity ~ $2.2k
24h volume ~ $0 at captured snapshot
```

Status:

```text
MARKET_VERIFIED
WATCH
QUOTE_REQUIRED
```

Source:
- https://www.geckoterminal.com/eth/pools/0x9628105808292699874f20d77d50a09bc26850c5

This pool historically had much larger liquidity, demonstrating that pool eligibility must be re-evaluated continuously rather than encoded from old snapshots.

---

## 8.3 Uniswap V4 — ALPH/ETH 2.9%

V4 uses a singleton PoolManager architecture; the identifier below is a **pool id**, not a standalone V3-style pool contract address.

```text
Pool ID
0xf0cdd5f94e3ef41a04f9cf19e933ad6ab03b2f759f8b5535370be7cff8986b92

Pair
ALPH / native ETH

Fee
2.90% = 290 bps
```

Current snapshot at research time:

```text
Liquidity ~ $6.8k
24h volume ~ $4.25k
38 transactions
```

Status:

```text
MARKET_VERIFIED
WATCH_HIGH_FEE
NOT_DEFAULT_EXECUTION_ROUTE
```

Source:
- https://www.geckoterminal.com/eth/pools/0xf0cdd5f94e3ef41a04f9cf19e933ad6ab03b2f759f8b5535370be7cff8986b92

The market is active, but a 290 bps LP fee on one leg creates a very high gross-spread hurdle before gas and the opposite leg.

---

## 8.4 Other Ethereum V4 pools discovered

### ALPH/ETH — 2.95%

```text
Pool ID
0xa03b4945e30c86d78b907801d4cfc6ade4c7ba5a4c5dd201dadc7ace4fe321e4

Fee
2.95%
```

Status: `WATCH_HIGH_FEE`.

### ALPH/USDT — 40%

```text
Pool ID
0xf90aea138d927b1e1bb2fc0edc7f07d4278317d747919e14ec3637c78056c06f

Fee
40%
```

Current snapshot showed only about ~$1.2k liquidity and no 24h volume.

Status:

```text
REJECT_FEE
```

Sources:
- https://www.geckoterminal.com/eth/pools/0xa03b4945e30c86d78b907801d4cfc6ade4c7ba5a4c5dd201dadc7ace4fe321e4
- https://www.geckoterminal.com/eth/pools/0xf90aea138d927b1e1bb2fc0edc7f07d4278317d747919e14ec3637c78056c06f

Conclusion: **pool discovery must not imply pool execution.** Some technically valid pools are economically unusable.

---

# 9. BNB Chain market map

Canonical wrapped asset:

```text
wALPH
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8
18 decimals
```

The two PancakeSwap routes originally supplied for this project are valid historical/official markets, but their **current liquidity state is critical**.

---

## 9.1 PancakeSwap V3 — ALPH/WBNB

```text
Pool
0xb685dF3cEC9E01048553355e9256267b1bD56E0E

ALPH
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8

WBNB
0xbb4CdB9CBd36B01bD1cBaEBF2De08d9173bc095c

Fee
1.00% = 100 bps
```

**Current research-date snapshot:**

```text
Liquidity ~ $0.07
24h volume $0
~0.5924 ALPH in pool snapshot
~0.00005485 WBNB in pool snapshot
```

Status:

```text
MARKET_VERIFIED
REJECT_LIQUIDITY
NOT EXECUTION ELIGIBLE
```

Source:
- https://www.geckoterminal.com/bsc/pools/0xb685df3cec9e01048553355e9256267b1bd56e0e

This is a major correction to the initial project assumption: **a valid PancakeSwap URL does not currently provide usable ALPH/WBNB arbitrage liquidity.**

---

## 9.2 PancakeSwap V3 — ALPH/USDT

```text
Pool
0xc44b6f04696bc502a27E90abCbf3a32f0DEFC29b

ALPH
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8

USDT
0x55d398326f99059fF775485246999027B3197955

Fee
1.00% = 100 bps
```

**Current research-date snapshot:**

```text
Liquidity ~ $8.75
24h volume $0
```

Status:

```text
MARKET_VERIFIED
REJECT_LIQUIDITY
NOT EXECUTION ELIGIBLE
```

Source:
- https://www.geckoterminal.com/bsc/pools/0xc44b6f04696bc502a27e90abcbf3a32f0defc29b

The displayed pool price is meaningless for our arbitrage engine at this depth. Even very small trades can produce pathological output.

---

## 9.3 Discovered BSC Uniswap V4 — ALPH/USDT 30%

A separate market exists on BSC outside PancakeSwap:

```text
Venue
Uniswap V4 (BSC)

Pool ID
0x1af86a9bff512be56a777c05238d6c3f4462aafe80927f0feb6d8b549d5bf8a4

Pair
ALPH / USDT

Fee
30%
```

Current research-date snapshot:

```text
Liquidity ~ $1.5k
24h volume ~ $83
```

Status:

```text
DISCOVERED
REJECT_FEE
```

Source:
- https://www.geckoterminal.com/bsc/pools/0x1af86a9bff512be56a777c05238d6c3f4462aafe80927f0feb6d8b549d5bf8a4

This pool must not be selected merely because its headline liquidity exceeds the current Pancake pools.

---

# 10. Immediate BSC conclusion

As of the 2026-09-05 research snapshot:

```text
Pancake ALPH/WBNB  -> REJECT_LIQUIDITY
Pancake ALPH/USDT  -> REJECT_LIQUIDITY
BSC Uniswap V4     -> REJECT_FEE
```

Therefore:

> **BNB Chain should currently be observer-only, not pre-funded as an active arbitrage venue, until a live market with acceptable executable depth reappears.**

This is not a permanent conclusion. Liquidity can migrate quickly. The bot/research collector must periodically rediscover and requote BSC markets.

This finding also means the initial idea:

```text
1,000 ALPH-equivalent Alephium
1,000 ALPH-equivalent Ethereum
1,000 ALPH-equivalent BSC
```

should **not** be treated as an automatic initial capital allocation. Capital should only be placed on execution-eligible venues.

---

# 11. CEX discovery — future extension, not active execution scope

Current market aggregators show native ALPH/USDT order books on several CEXs.

Research-date discovery set:

| CEX | Pair | Observed condition | Current research status |
|---|---|---|---|
| MEXC | ALPH/USDT | highest/one of highest reported ALPH spot volumes | `DISCOVERY_ONLY` |
| Bitget | ALPH/USDT | active order book | `DISCOVERY_ONLY` |
| Gate | ALPH/USDT | active order book | `DISCOVERY_ONLY` |
| CoinEx | ALPH/USDT | active but thinner | `DISCOVERY_ONLY` |
| NonKYC.io | ALPH/USDT | wide spread / zero ±2% depth in observed snapshot | `REJECT_DISCOVERY` |

CoinGecko snapshots during this research showed CEX volumes materially larger than the currently visible DEX volume, but order-book depth changes second-by-second. The aggregate snapshot is useful for discovery only.

Source:
- https://www.coingecko.com/en/coins/alephium

Before any CEX becomes execution eligible we still need to verify:

```text
native ALPH deposit network
native ALPH withdrawal network
withdrawal fee
withdrawal minimum
maintenance state
confirmation requirements
API/WebSocket quality
order types
maker/taker fees
regional/account constraints
live orderbook depth by size
```

No CEX is active scope yet.

---

# 12. Current market graph

The current evidence graph is therefore closer to:

```text
                         EconomicAsset: ALPH
                                │
          ┌─────────────────────┼──────────────────────┐
          │                     │                      │
          ▼                     ▼                      ▼
      ALEPHIUM               ETHEREUM                 BSC
   native ALPH            wrapped ALPH            wrapped ALPH
          │                     │                      │
   ┌──────┼──────┐        ┌─────┼──────┐        ┌──────┼──────┐
   ▼      ▼      ▼        ▼     ▼      ▼        ▼             ▼
Elexium  AYIN  Nightshade V3   V3     V4       Pancake V3   Uni V4
 factory  pools registry   │     │      │          │             │
                           │     │      │          │             │
                       ALPH/  ALPH/  ALPH/ETH   near-zero      30% fee
                       USDT   WETH   high fee    liquidity

                     Future / discovery-only
                              │
                 ┌────────────┼────────────┐
                 ▼            ▼            ▼
                MEXC        Bitget        Gate ...
```

Current likely research priority:

```text
1. Native Alephium live executable routes
2. Ethereum Uniswap V3/V4 live quote ladder
3. Cross-venue Alephium <-> Ethereum economics
4. Continue observing BSC for liquidity return
5. CEX extension later
```

---

# 13. Trading graph vs transfer graph

Do not confuse these.

## Trading graph

```text
Venue inventory A
      │
      │ buy/sell
      ▼
Economic exposure
      │
      │ opposite leg
      ▼
Venue inventory B
```

## Transfer graph

```text
Alephium wallet
      │
      │ bridge
      ▼
Ethereum wallet
```

or later:

```text
Alephium wallet
      │
      │ CEX deposit
      ▼
CEX account
```

A trading edge can exist even when the transfer edge is slow because the strategy is prefunded.

A transfer edge is relevant to:

- inventory recovery,
- capital recycling,
- rebalance latency,
- rebalance cost,
- route availability.

---

# 14. Market record schema — revised

Every market should eventually be represented as:

```text
MarketRecord {
    market_id

    chain
    venue
    venue_type
    protocol_version

    discovery_type
    factory_or_registry
    pool_address_or_pool_id

    base_settlement_asset
    quote_settlement_asset
    base_economic_asset
    quote_economic_asset

    decimals_base
    decimals_quote

    pool_type
    fee_model
    fee_tier

    router_or_execution_entrypoint

    discovered_from
    identity_verified_from

    snapshot_block
    snapshot_timestamp

    headline_liquidity
    headline_volume_24h

    last_quote_timestamp
    quoteable
    simulatable

    status
    rejection_reason
}
```

For V4-style singleton pools:

```text
pool_address_or_pool_id
```

must preserve the distinction between `PoolManager address` and `PoolId`.

---

# 15. Execution eligibility ladder

Market discovery should progress through gates:

```text
L0  ASSET IDENTITY
    ↓
L1  MARKET EXISTS
    ↓
L2  FACTORY / POOL / MARKET ATTRIBUTION
    ↓
L3  FEE MODEL KNOWN
    ↓
L4  LIVE QUOTE WORKS
    ↓
L5  QUOTE LADDER FOR OUR SIZE WORKS
    ↓
L6  SIMULATION / PREFLIGHT WORKS
    ↓
L7  GAS + EXECUTION COST KNOWN
    ↓
L8  ROUTE PASSES ECONOMIC THRESHOLD
    ↓
EXECUTION ELIGIBLE
```

Indexer liquidity is not a substitute for L4–L8.

---

# 16. Standard quote ladder required by this research

For every surviving market/route:

```text
5 ALPH
10 ALPH
20 ALPH
40 ALPH
75 ALPH
100 ALPH
125 ALPH
150 ALPH
```

Capture both directions whenever possible:

```text
ALPH -> quote
quote -> exact/approximately equivalent ALPH
```

Required fields:

```text
timestamp
block/state id
venue
pool/route
size
amount_in
amount_out
effective_price
fee tier
estimated gas
price impact
quote age
```

The market map cannot be called complete until this ladder exists for all candidate active routes.

---

# 17. Major findings from research pass 1

## Finding A — the original BSC assumption is currently false

The official/known PancakeSwap pools exist, but the current ALPH/WBNB and ALPH/USDT liquidity is effectively unusable.

Therefore:

```text
market exists != arbitrage venue exists
```

## Finding B — Ethereum liquidity also migrated materially

Historical snapshots of the same Uniswap V3 pools showed much larger liquidity than recent snapshots. Venue selection must be live and dynamic.

## Finding C — Uniswap V4 adds markets but not necessarily usable markets

Some currently active ALPH V4 pools charge 2.9%–2.95%, and an ALPH/USDT pool is observed at 40% fee. Discovery without fee validation would create catastrophic false opportunities.

## Finding D — Alephium native market discovery can be deterministic

We have current factory/registry anchors for:

```text
Elexium
AYIN Classic + CL
Nightshade
```

so native discovery does not need to depend on UI/indexer scraping.

## Finding E — quote provenance matters

`USDTeth` and `USDTbsc` are different settlement assets. A route graph that collapses them into `USDT` will produce invalid inventory and rebalance assumptions.

## Finding F — CEX may later be strategically important

Current aggregate data shows ALPH CEX spot activity significantly exceeds many current DEX pools. CEX support should remain architectural future scope even though it is not active implementation scope yet.

---

# 18. What remains unresolved

The following are blockers before `01_market_map.md` can become `VERIFIED`.

## Native Alephium

- [ ] enumerate every current Elexium pool from Pair Factory
- [ ] identify all Elexium pools containing ALPH
- [ ] record pool contract id/address, paired token, pool type and actual fee
- [ ] enumerate every AYIN CL sub-pool from CL factory
- [ ] map ALPH-containing AYIN CL pools + fee tiers
- [ ] enumerate Nightshade pair registry and map ALPH markets
- [ ] identify authoritative router/execution entrypoints for each venue
- [ ] obtain live quote ladder for every viable ALPH route

## Ethereum

- [x] verify canonical wALPH
- [x] identify major V3 ALPH/USDT pool
- [x] identify major V3 ALPH/WETH pool
- [x] identify currently visible V4 ALPH markets
- [x] verify fee tiers for mapped EVM pools
- [ ] obtain direct on-chain quotes at standard sizes
- [ ] map router/path choice and gas by route
- [ ] identify any additional economically relevant pools not surfaced by current indexers

## BSC

- [x] verify canonical wALPH
- [x] verify PancakeSwap ALPH/WBNB address + fee
- [x] verify PancakeSwap ALPH/USDT address + fee
- [x] measure current indexer liquidity condition
- [x] classify both Pancake pools as currently non-executable
- [ ] continuously monitor for liquidity migration / new pools
- [ ] discover any lower-fee ALPH pools that become economically viable

## CEX — future

- [x] identify current discovery candidates
- [ ] verify ALPH native deposit/withdraw networks
- [ ] verify fees/minimums/maintenance status
- [ ] capture live websocket orderbooks
- [ ] measure executable depth at standard sizes

---

# 19. Current execution-universe classification

This is **not** a production allowlist. It is the current research classification.

| Market | Status | Reason |
|---|---|---|
| AYIN ALPH/USDTeth Classic | `QUOTE_REQUIRED` | pool identity verified; live depth pending |
| AYIN ALPH/USDCeth Classic | `QUOTE_REQUIRED` | pool identity verified; live depth pending |
| AYIN ALPH/WETH Classic | `QUOTE_REQUIRED` | pool identity verified; live depth pending |
| AYIN CL pools | `DISCOVERY_VERIFIED` | factory known; ALPH child-pools need enumeration |
| Elexium ALPH routes | `DISCOVERY_VERIFIED` | factory known; dynamic pool enumeration pending |
| Nightshade ALPH routes | `DISCOVERY_VERIFIED` | registry known; dynamic pair enumeration pending |
| Uniswap V3 ALPH/USDT | `QUOTE_REQUIRED` | valid 1% pool; recent liquidity still potentially relevant |
| Uniswap V3 ALPH/WETH | `WATCH` | valid 1% pool; recent liquidity much thinner |
| Uniswap V4 ALPH/ETH 2.9% | `WATCH_HIGH_FEE` | active but 290 bps LP fee |
| Uniswap V4 ALPH/ETH 2.95% | `WATCH_HIGH_FEE` | high fee + thinner liquidity |
| Uniswap V4 ALPH/USDT 40% | `REJECT_FEE` | economically unsuitable for normal arbitrage |
| Pancake V3 ALPH/WBNB | `REJECT_LIQUIDITY` | ~$0.07 observed liquidity |
| Pancake V3 ALPH/USDT | `REJECT_LIQUIDITY` | ~$8.75 observed liquidity |
| BSC Uni V4 ALPH/USDT 30% | `REJECT_FEE` | 3000 bps fee |
| MEXC / Bitget / Gate / CoinEx | `DISCOVERY_ONLY` | future CEX phase; transfer/API checks pending |

---

# 20. Research output files

Measured snapshots belong under:

```text
reserch/data/pools/
```

The first normalized snapshot associated with this pass should contain at minimum:

```text
snapshot_date
chain
venue
protocol_version
market
pool_or_id
base_asset
quote_asset
fee_bps
headline_liquidity_usd
headline_volume_24h_usd
evidence_status
source
notes
```

Raw evidence must preserve its timestamp because liquidity and volume are ephemeral.

---

# 21. Acceptance criteria

`01_market_map.md` becomes `VERIFIED` only when all of the following are true:

```text
[ ] canonical ALPH settlement assets mapped
[x] canonical EVM wrapped ALPH identities mapped
[x] major current EVM pools discovered
[x] native Alephium venue factory/registry anchors mapped
[ ] all native ALPH pools enumerated from chain
[ ] pool fee/model known for every candidate
[ ] standard-size quote ladder collected
[ ] dead/spoof/high-fee pools rejected automatically
[ ] route universe can be rebuilt from factories/registries/APIs
[ ] no critical market identity depends only on memory or a UI URL
```

Current overall status:

```text
PARTIALLY_VERIFIED
```

The biggest remaining blocker is **native Alephium pool enumeration + live executable quote capture**.

---

# 22. Source hierarchy

Prefer evidence in this order:

```text
1. on-chain state / canonical chain API
2. official protocol contracts / factory / registry
3. official Alephium token + bridge documentation
4. official DEX documentation
5. verified explorer data
6. protocol adapters that read on-chain state (e.g. DefiLlama)
7. market indexers (GeckoTerminal/CoinGecko) for discovery/snapshots
8. UI links only as hints
```

Never elevate an indexer or UI value above a fresh executable quote.

---

## Primary references used in this pass

- Alephium overview / current venue universe: https://docs.alephium.org/
- Alephium bridge docs: https://docs.alephium.org/infrastructure/the-bridge/
- Bridge/token verification: https://alephium.org/news/post/verification-of-bridge-contracts-tokens-token-lists-76e5c237bf52/
- Alephium token list: https://github.com/alephium/token-list/blob/master/tokens/mainnet.json
- Elexium docs: https://docs.elexium.finance/
- AYIN Smart Router: https://docs.ayin.app/ayin/ayin-metaswap/what-is-smart-router
- AYIN Concentrated Liquidity: https://docs.ayin.app/ayin/ayin-metadex/concentrated
- Nightshade: https://nightshade.finance/token/ALPH
- DefiLlama Elexium adapter: https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/elexium/index.js
- DefiLlama AYIN adapter: https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/ayin/index.js
- DefiLlama Nightshade adapter: https://github.com/DefiLlama/DefiLlama-Adapters/blob/main/projects/nightshade/index.js
- Ethereum wALPH: https://etherscan.io/token/0x590f820444fa3638e022776752c5eef34e2f89a6
- BSC wALPH: https://bscscan.com/token/0x8683ba2f8b0f69b2105f26f488bade1d3ab4dec8
- Uniswap V3 ALPH/USDT: https://www.geckoterminal.com/eth/pools/0xa344855388c9f2760e998eb2207b58de6e7d0360
- Uniswap V3 ALPH/WETH: https://www.geckoterminal.com/eth/pools/0x9628105808292699874f20d77d50a09bc26850c5
- Uniswap V4 ALPH/ETH 2.9%: https://www.geckoterminal.com/eth/pools/0xf0cdd5f94e3ef41a04f9cf19e933ad6ab03b2f759f8b5535370be7cff8986b92
- Pancake V3 ALPH/WBNB: https://www.geckoterminal.com/bsc/pools/0xb685df3cec9e01048553355e9256267b1bd56e0e
- Pancake V3 ALPH/USDT: https://www.geckoterminal.com/bsc/pools/0xc44b6f04696bc502a27e90abcbf3a32f0defc29b
- CoinGecko current ALPH markets: https://www.coingecko.com/en/coins/alephium
