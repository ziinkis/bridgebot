# 05 — Executable Liquidity

## Objective

Mengukur berapa banyak ALPH yang benar-benar dapat diperdagangkan pada setiap venue/route tanpa mengandalkan TVL atau displayed spot price sebagai proxy.

## Core principle

```text
DISPLAYED LIQUIDITY != EXECUTABLE LIQUIDITY
```

Terutama untuk concentrated-liquidity AMMs, total pool value tidak langsung memberi tahu hasil trade size tertentu.

## Standard quote ladder

Starting measurement ladder:

```text
5 ALPH
10
20
40
60
75
100
125
150
```

Jika liquidity lebih dalam, ladder dapat diperluas. Jika sangat tipis, tambahkan bucket lebih kecil.

## Required observation

```text
QuoteObservation {
    timestamp
    chain
    venue
    pool_or_route

    asset_in
    asset_out
    amount_in
    amount_out

    effective_price
    fee_tiers[]
    estimated_gas

    block_or_state
    quote_method
    success
    error_if_any
}
```

## Bidirectional measurement

Setiap route harus diukur dua arah jika relevan:

```text
ALPH → quote
quote → ALPH
```

Jangan menyimpulkan sell depth dari buy depth.

## Multi-route comparison

Untuk size yang sama, capture seluruh candidate route yang masuk akal.

Example:

```text
ALPH → USDT direct
ALPH → WETH → USDT
```

Store both, then select by net executable economics.

## Size optimizer input

Liquidity research harus menghasilkan function empiris:

```text
q -> expected_amount_out(q)
```

Dari dua venue, economics engine dapat mencari:

```text
q* = argmax economic_net(q)
```

## Staleness

Liquidity observations adalah snapshots.

Setiap row wajib mempunyai timestamp + chain state reference.

Do not write:

```text
pool supports 100 ALPH safely
```

sebagai fakta permanen hanya dari satu snapshot.

## Research tasks

- [ ] collect quote ladders Elexium
- [ ] collect quote ladders AYIN
- [ ] collect executable Nightshade paths if suitable
- [ ] collect Uniswap ladders
- [ ] collect PancakeSwap ladders
- [ ] capture bidirectional quotes
- [ ] capture direct vs routed alternatives
- [ ] repeat across time windows
- [ ] calculate success/revert/quote availability rate

## Output

Primary raw data:

```text
reserch/data/quotes/
```

## Acceptance criteria

Kita dapat membuat table untuk setiap venue:

```text
size
→ amountOut
→ effective price
→ estimated gas
→ route
```

dan menentukan maximum economically useful size berdasarkan data, bukan asumsi.
