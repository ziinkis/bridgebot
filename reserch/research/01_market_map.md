# 01 — Market Map

## Objective

Membangun peta lengkap semua lokasi dan route yang secara nyata dapat memperdagangkan **EconomicAsset = ALPH**.

Research ini harus menjawab:

```text
WHERE can ALPH be traded?
WITH WHAT settlement asset?
THROUGH WHICH pool/market/router?
AT WHAT fee tier?
WITH HOW MUCH executable depth?
```

## Initial venue universe

### Alephium

Candidate venues:

- Elexium
- AYIN
- Nightshade
- venue lain hanya setelah contract/factory dan liquidity terverifikasi

Potential quote assets yang perlu dipetakan:

- native/stable bridged assets
- USDTeth
- USDTbsc
- other economically relevant routes

### Ethereum

- Uniswap
- canonical wALPH contract must be verified
- direct and routed paths must be enumerated

### BNB Chain

- PancakeSwap
- canonical wALPH contract must be verified
- direct and routed paths must be enumerated

### Future CEX

Do not add to active scope yet, but market map schema must support:

```text
CEX
market symbol
base/quote
orderbook depth
withdraw/deposit networks
```

## Required pool/market record

```text
MarketRecord {
    chain
    venue
    venue_type
    factory_or_market_id
    pool_or_market_address

    asset0
    asset1
    economic_asset0
    economic_asset1

    decimals0
    decimals1

    pool_type
    fee_tier
    router

    discovered_from
    verified_from

    block_or_snapshot
    timestamp

    status
}
```

## Verification rule

A pool/market is not execution-eligible merely because it appears in a UI or indexer.

Required checks:

```text
1. token identity verified
2. chain verified
3. canonical factory/venue verified
4. pool/market exists on-chain or authoritative venue API
5. fee configuration identified
6. non-zero and usable executable liquidity
7. route can be quoted
8. route can be simulated/preflighted
```

## Route graph

Do not hardcode only:

```text
Alephium ↔ Ethereum
Alephium ↔ BSC
Ethereum ↔ BSC
```

Instead construct graph from actual venue edges.

Example conceptual graph:

```text
                      ECONOMIC ALPH
                           │
            ┌──────────────┼──────────────┐
            │              │              │
            ▼              ▼              ▼
       ALEPHIUM          ETHEREUM         BSC
            │              │              │
      Elexium/AYIN      Uniswap       PancakeSwap
            │              │              │
       pool routes      pool routes      pool routes
```

## Research tasks

- [ ] enumerate canonical ALPH/wALPH settlement assets
- [ ] enumerate Alephium pools containing ALPH
- [ ] enumerate Ethereum wALPH pools
- [ ] enumerate BSC wALPH pools
- [ ] capture pool type and fee tier
- [ ] identify canonical routers
- [ ] identify route alternatives/multi-hop paths
- [ ] reject spoof/non-canonical token pools
- [ ] snapshot executable quotes for standard sizes
- [ ] tag stale/dead/illiquid markets

## Output

Primary measured output belongs in:

```text
reserch/data/pools/
```

Suggested normalized CSV/JSON columns:

```text
timestamp
chain
venue
pool
pool_type
asset_in
asset_out
fee_tier
router
liquidity_reference
block_number
status
source
```

## Acceptance criteria

This research is `VERIFIED` only when route universe can be rebuilt from evidence without relying on memory or hardcoded UI assumptions.
