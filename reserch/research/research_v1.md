# ALPH Cross-Chain Arbitrage Research v1

## Scope

Baseline research untuk desain **ALPH multi-venue inventory arbitrage engine** dengan initial focus:

- Alephium native ALPH
- Alephium DEX
- Ethereum / Uniswap
- BNB Chain / PancakeSwap
- Alephium Bridge
- future CEX extension
- maximum initial capital: **1,000 ALPH-equivalent per active venue**

Dokumen ini adalah **BASELINE**, bukan final architecture specification. Semua angka yang berubah dengan waktu harus divalidasi ulang dengan live/on-chain data sebelum digunakan untuk execution.

---

# 1. Core thesis

Model yang sedang diuji:

```text
PREFUNDED INVENTORY
        +
MULTI-VENUE EXECUTABLE QUOTES
        +
DYNAMIC SIZE OPTIMIZATION
        +
INVENTORY-AWARE ARBITRAGE
        +
NATURAL REVERSE FLOW
        +
FLOW NETTING
        +
MIN-COST REBALANCING
```

Bridge tidak ditempatkan di hot path setiap trade.

Bad model:

```text
detect
→ buy
→ bridge
→ wait
→ sell
```

Preferred model:

```text
detect
→ quote both prefunded venues
→ size
→ risk check
→ execute both legs near-concurrently
→ reconcile inventory
→ net future flows
→ rebalance only when economically required
```

---

# 2. Asset identity

Canonical wALPH addresses yang menjadi baseline research:

```text
Ethereum
0x590F820444fA3638e022776752c5eEF34E2F89A6

BNB Chain
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8
```

Jangan menggunakan ticker sebagai unique asset identity.

Conceptual model:

```text
EconomicAsset = ALPH

SettlementAsset examples:
- ALPH_NATIVE_ALEPHIUM
- WALPH_ETHEREUM
- WALPH_BSC
- future ALPH_CEX_LOCATION
```

Stablecoin provenance juga harus dibedakan:

```text
USDTeth != USDTbsc
```

meskipun keduanya dapat memiliki economic valuation yang dekat dengan USD.

---

# 3. Fee baseline

Initial research menemukan ALPH routes dengan fee yang cukup besar.

Examples yang harus selalu divalidasi terhadap current pool state:

```text
Uniswap V3 ALPH pools observed: 1.00% fee tier
PancakeSwap V3 ALPH paths observed: 1.00% ALPH pool fee tier
Elexium documented standard: ~0.30%, configurable
AYIN: multiple pool types / possible fee tiers
```

Pure multiplicative LP-fee illustration:

```text
Elexium 0.30% ↔ Uniswap 1.00%

1 - (0.997 × 0.99)
= 1.297%
= 129.7 bps
```

```text
Uniswap 1.00% ↔ Pancake 1.00%

1 - 0.99²
= 1.99%
= 199 bps
```

Karena itu `spot_spread > fixed_bps` tidak cukup untuk menentukan execution.

---

# 4. Executable quote is the source of truth

Jangan membandingkan displayed price.

Gunakan object semacam:

```text
ExecutableQuote {
    chain
    venue
    pool_or_market
    route
    fee_tiers[]

    amount_in
    expected_amount_out

    gas_estimate
    block_or_state_reference
    timestamp
    quote_age

    estimated_price_impact
}
```

Untuk AMM quoter, expected output umumnya sudah merepresentasikan LP fee dan deterministic curve impact pada state yang digunakan.

Jangan mengurangi fee/impact dua kali.

---

# 5. Price impact vs slippage

```text
PRICE IMPACT
= deterioration karena size kita pada liquidity curve saat quote dibuat
```

```text
SLIPPAGE / QUOTE DRIFT
= perubahan antara expected quote dan actual execution karena market/pool state bergerak setelah quote
```

Fixed slippage seperti 0.5% atau 1% berbahaya untuk arbitrage karena dapat mengizinkan transaksi tetap sukses meski seluruh edge sudah hilang.

Preferred rule:

```text
slippage budget
<=
expected edge
- minimum required final profit
- execution risk reserve
```

Long term, slippage allowance harus dikalibrasi dari measured adverse quote drift p95/p99 per venue/route/size bucket.

---

# 6. Profit model

Conceptual hot-path economics:

```text
HOT_PATH_NET(q) =

SELL_PROCEEDS(q)
-
BUY_COST(q)
-
GAS_BUY(q)
-
GAS_SELL(q)
-
FAILED_LEG_RESERVE(q)
-
MEV_OR_ADVERSE_SELECTION_RESERVE(q)
```

Economic result:

```text
ECONOMIC_NET(q) =

HOT_PATH_NET(q)
-
DELTA_REBALANCE_LIABILITY(q)
-
INVENTORY_SHADOW_COST(q)
```

And:

```text
NET_BPS(q) =
ECONOMIC_NET(q)
/ CAPITAL_AT_RISK(q)
× 10,000
```

---

# 7. Size optimization

Do not use one fixed trade size.

Initial research ladder:

```text
5
10
20
40
60
75
100
125
150 ALPH
```

The optimizer searches:

```text
q* = argmax ECONOMIC_NET(q)
```

subject to:

```text
inventory_after_safe
max_single_failure_loss_safe
quote_fresh
route_valid
minimum_profit_satisfied
```

For initial capital 1,000 ALPH-equivalent/venue, a conservative initial operational exposure around 75–100 ALPH may be tested, but this remains an **ASSUMPTION** until quote and failure data exist.

---

# 8. Inventory is multi-dimensional

Starting conceptual neutral allocation:

```text
~500 ALPH/wALPH
~500 quote-equivalent
```

This is not final.

Per venue inventory must track more than base balance:

```text
VenueInventory {
    base_asset
    quote_asset
    native_gas

    available
    reserved
    pending_out
    pending_in
    in_transit
    redeemable
}
```

Why?

Example:

```text
Start:
ALEPHIUM 500 ALPH + 500 quote
ETH      500 wALPH + 500 USDT

Trade:
BUY 100 ALPH Alephium
SELL 100 wALPH Ethereum

After:
ALEPHIUM 600 ALPH + 400 quote
ETH      400 wALPH + 600 USDT
```

Bridging 100 ALPH Alephium → ETH restores base inventory but does not restore quote inventory.

Therefore rebalance is a **multi-asset network-flow problem**.

---

# 9. Trade capacity is more useful than ratio alone

Ratio bands can be secondary safety controls.

Primary metric should include remaining executable trade capacity.

Example:

```text
p95 normal trade size = 75 ALPH
available sell inventory = 450 ALPH

remaining sell capacity ≈ 6 trades
```

This adapts automatically if optimal trade size changes.

---

# 10. Natural flow and netting

Best rebalance can be another profitable arbitrage in the opposite direction.

Example:

```text
Trade A drift: +100 ALPH
Trade B reverse drift: -80 ALPH

net imbalance = 20 ALPH
```

Do not bridge 100 one way then 80 back.

Priority:

```text
1. NATURAL REVERSE ARBITRAGE
2. FLOW NETTING
3. WAIT WHILE SAFE
4. PARTIAL REBALANCE
5. BATCH BRIDGE / TRANSFER
6. EMERGENCY REBALANCE
```

---

# 11. Bridge decision

Bridge should not trigger after every trade.

Main principle:

```text
BRIDGE WHEN
EXPECTED COST OF NOT REBALANCING
>
EXPECTED COST OF REBALANCING
```

The decision must consider:

- current usable inventory
- pending transfers
- predicted directional flow
- remaining trade capacity
- bridge health
- source/destination gas
- transfer amount
- fixed/variable bridge cost
- settlement latency distribution
- likely reverse arbitrage
- opportunity loss from inventory exhaustion

A better trigger than fixed percentage:

```text
if time_to_inventory_floor
<=
transfer_p95_latency + safety_margin
then rebalance candidate becomes urgent
```

---

# 12. Bridge batching

Fixed transfer costs become expensive in bps for small amounts.

Conceptual formula:

```text
bridge_cost_bps =
bridge_cost_in_common_value
/ transfer_amount_in_common_value
× 10,000
```

Minimum economic batch can be derived from:

```text
Q_MIN =
FIXED_TRANSFER_COST × 10,000
/ MAX_ACCEPTABLE_TRANSFER_BPS
```

Parameter values must come from measured current bridge costs, not hardcoded examples.

---

# 13. Bridge state machine

Do not treat submit as settlement.

```text
PLANNED
→ RESERVED
→ ALLOWANCE_CHECK
→ SOURCE_SIMULATED
→ SOURCE_SUBMITTED
→ SOURCE_CONFIRMED
→ FINALITY_WAIT
→ VAA_PENDING
→ VAA_VERIFIED
→ DESTINATION_SIMULATED
→ REDEEM_SUBMITTED
→ REDEEM_CONFIRMED
→ BALANCE_RECONCILED
→ SETTLED
```

Error/recovery states must cover:

```text
BRIDGE_PAUSED
SOURCE_FAILED
SOURCE_REORG
VAA_TIMEOUT
REDEEM_FAILED
BALANCE_MISMATCH
ACCOUNTING_MISMATCH
```

Pending/in-transit balance is not tradable inventory.

---

# 14. Bridge health

Bridge health must be first-class risk state.

Do not define healthy as only `HTTP 200` from one service.

Research should cover:

```text
source chain progress
destination chain progress
contract paused state
recent transfer success
VAA progression
VAA latency distribution
redeem simulation
backlog / stuck transfers
```

If bridge is down but prefunded inventories remain healthy, trading can potentially continue inside stricter inventory limits; directions that worsen a shortage should eventually be disabled.

---

# 15. Gas

Alephium, Ethereum, and BSC gas must be estimated per actual route.

Do not hardcode one gas amount for every swap.

Track:

```text
estimated gas
actual gas
effective gas price
gas estimation error
route type
size bucket
```

Native gas reserve must be separated from trading inventory.

The bot must retain enough native gas for normal swaps plus emergency unwind and settlement/redeem actions.

---

# 16. UTXO / chain-specific operational risk

Alephium is UTXO-based. Long-running execution can create fragmentation.

Research/implementation must eventually monitor:

```text
UTXO count
fragmentation
dust
transaction-build complexity
```

Consolidation should be an idle/maintenance operation, not something triggered during an active opportunity.

---

# 17. Cross-chain execution is not atomic

Core failure scenario:

```text
LEG A SUCCESS
LEG B FAIL
```

Before opening the trade, compute an emergency unwind route and approximate loss.

Execution flow should approach:

```text
RESERVE BOTH INVENTORIES
→ REFRESH BOTH QUOTES
→ SIMULATE / PREFLIGHT
→ BUILD / SIGN
→ NEAR-CONCURRENT SUBMIT
→ TRACK EACH LEG
→ RECONCILE
→ UNWIND/HEDGE IF REQUIRED
```

`max_single_failure_loss` is a hard safety constraint, not merely an expected-value adjustment.

---

# 18. Quote-drift learning

Store quote snapshots after detection even when no trade is executed:

```text
t0
t0 + 1s
t0 + 2s
next relevant block
expected inclusion window
```

Build adverse drift distributions:

```text
p50
p90
p95
p99
```

per:

```text
venue
route
size bucket
market regime
```

This data becomes the basis for dynamic slippage and execution risk buffers.

---

# 19. Inventory shadow pricing

An opportunity that worsens scarce inventory should face an inventory penalty.

A trade that avoids an otherwise necessary rebalance may receive economic credit.

But:

```text
inventory_credit(q)
<=
R(before) - R(after)
```

where `R()` is the estimated minimum future rebalance cost.

Do not manufacture artificial profit by giving oversized inventory credits.

---

# 20. Rebalancing as min-cost flow

Conceptual nodes:

```text
(chain/location, settlement asset)
```

Possible edges:

```text
DEX swap
bridge
native transfer
future CEX deposit
future CEX withdrawal
wait
natural expected flow
```

Each edge has:

```text
cost
capacity
latency
risk
availability
```

Optimization objective:

```text
minimize
fee + gas + price impact + latency cost + risk cost

subject to
minimum required inventory capacity at active venues
```

---

# 21. Future CEX extension

Do not make `DEX` the core abstraction.

Use conceptual `Venue`:

```text
VENUE
├── DEX
└── CEX
```

And generic `Transfer`:

```text
TRANSFER
├── bridge
├── native chain transfer
├── CEX deposit
└── CEX withdrawal
```

For CEX, executable quote must walk orderbook depth; best bid/ask is not sufficient for a requested size.

CEX execution introduces:

- maker/taker fee
- orderbook impact
- partial fill
- IOC/FOK behavior
- websocket sequence integrity
- custody/location risk
- withdrawal fee/status/limits

The same prefunding principle remains useful: CEX withdrawal should normally be settlement/rebalancing, not hot-path arbitrage dependency.

---

# 22. Required telemetry

From first experiments, preserve:

```text
detection_spread_bps
quoted_edge_bps
executed_edge_bps

pool_fee_bps
estimated_price_impact_bps
quote_drift_bps

gas_estimated
gas_actual

trade_size
leg_a_latency
leg_b_latency
leg_success
leg_failure
unwind_cost

inventory_before
inventory_after
rebalance_liability_before
rebalance_liability_after

bridge_cost
bridge_latency
vaa_latency

realized_pnl
economic_pnl
```

---

# 23. PnL layers

Store separately:

```text
GROSS_ARB_PNL
HOT_PATH_NET_PNL
ECONOMIC_NET_PNL
```

`ECONOMIC_NET_PNL` is the final research metric because it includes the future cost created by inventory imbalance.

---

# 24. Initial experimental configuration

These are assumptions for measurement, not final decisions:

```text
capital per active venue: <= 1,000 ALPH-equivalent
initial base/quote split: ~50/50
candidate sizes: 5/10/20/40/60/75/100/125/150 ALPH
initial normal size cap to test: ~75–100 ALPH
```

All must be replaced or confirmed by data.

---

# 25. Next validation sequence

1. Enumerate every relevant ALPH pool/market and contract.
2. Verify settlement asset identities and decimals.
3. Verify fee tier/config for every candidate pool.
4. Capture executable quote ladders for standard size buckets.
5. Measure gas per route.
6. Build price-impact curves.
7. Collect quote-drift distributions.
8. Measure bridge costs and state transitions.
9. Measure bridge latency p50/p95/p99 per direction.
10. Model base + quote + gas inventory capacity.
11. Simulate natural netting vs bridge frequency.
12. Estimate failed-leg and unwind loss.
13. Run full economic viability simulation before real-money implementation.

---

# Final baseline conclusion

Research v1 favors the following system thesis:

```text
ALPH MULTI-VENUE INVENTORY ARBITRAGE ENGINE
```

not a simple:

```text
BRIDGE ARBITRAGE BOT
```

The main question is not merely whether price differs between chains. The real question is:

> **After executable depth, fees, gas, quote drift, execution failure risk, inventory depletion, and future rebalancing are priced correctly, is there repeatable positive ECONOMIC_NET_PNL within the 1,000 ALPH-equivalent per-venue capital constraint?**

That question must be answered by the remaining research files and measured datasets.
