# 10 — Inventory Model

## Objective

Menentukan bagaimana bot membaca, membatasi, dan menggunakan inventory aktual di setiap venue agar dapat mengeksekusi opportunity tanpa menunggu transfer, sambil menjaga capital efficiency dan risk.

## Core rule: no fixed capital assumption

Tidak ada constraint desain seperti:

```text
MAX CAPITAL PER ACTIVE VENUE = 1,000 ALPH-equivalent
```

Angka `100`, `500`, `1,000`, `100,000 ALPH-equivalent`, atau nominal lain hanya boleh menjadi **research scenario**.

Runtime engine harus mengetahui modal melalui state aktual, bukan konstanta.

```text
actual balances
    ↓
subtract locked / reserved / pending / in-transit
    ↓
subtract native-gas and emergency reserve
    ↓
apply operator allocation ceiling when configured
    ↓
usable allocated inventory
```

## Wallet balance vs allocated capital

Jangan menyamakan:

```text
wallet_balance
```

dengan:

```text
bot_spendable_balance
```

Dua operating modes harus didukung:

### Dedicated execution wallet/account

Jika wallet/account hanya digunakan bot:

```text
usable_capital
≈
available_balance
- safety_reserves
```

### Shared wallet/account

Jika balance juga digunakan untuk tujuan lain:

```text
usable_capital
=
min(
    available_balance - safety_reserves,
    operator_allocation_ceiling
)
```

Bot tidak boleh menghabiskan dana di luar allocation ceiling.

## Inventory dimensions

Per venue/location:

```text
InventoryState {
    settlement_asset
    location

    onchain_or_account_balance
    available
    allocated
    reserved
    locked
    pending_out
    pending_in
    in_transit
    redeemable

    native_gas_reserve
    emergency_reserve

    common_value
}
```

Native gas reserve tracked separately but participates in venue health.

## Two-sided capacity

A venue needs both:

```text
base sell capacity
quote buy capacity
```

Nominal total capital tidak cukup untuk menjelaskan execution capacity.

Contoh:

```text
venue value = 10,000 ALPH-equivalent
base = almost all capital
quote = near zero
```

Venue tersebut tetap tidak mempunyai buy capacity yang memadai.

## Dynamic trade capacity

Primary inventory metric harus berasal dari actual usable balance dan executable trade-size distribution.

Example only:

```text
available base = 450 ALPH
p95 intended sell size = 75 ALPH

remaining sell capacity ≈ 6 p95 trades
```

Jika actual available base berubah menjadi `45 ALPH`, capacity berubah otomatis.
Jika menjadi `45,000 ALPH`, liquidity/risk constraints mungkin menjadi limiter sebelum balance.

## Dynamic sizing boundary

Untuk candidate trade:

```text
q_max = min(
    buy_side_usable_capacity,
    sell_side_usable_capacity,
    liquidity_safe_size,
    per_trade_risk_limit,
    venue_limit
)
```

Optimizer kemudian mencari:

```text
q* = argmax ECONOMIC_NET(q)
subject to 0 < q <= q_max
```

Implikasi:

```text
small capital
→ gas/fee dapat membuat seluruh route NO_TRADE

large capital
→ liquidity/impact/risk dapat membatasi q jauh di bawah total balance
```

Tidak ada kewajiban menggunakan persentase tertentu dari modal.

## Ratio bands are relative, not nominal

Base/quote ratio masih berguna sebagai secondary guard, tetapi exact bands belum final.

Possible research concepts:

```text
healthy region
warning region
critical region
stop-worsening direction
```

Semua threshold harus diturunkan dari:

```text
directional flow
trade size distribution
transfer latency
transfer cost
route reliability
```

bukan dari nominal `1,000 ALPH`.

## Inventory reservation

Before cross-venue execution:

```text
reserve both legs
```

Reserved balance cannot be used by another candidate.

This prevents overcommit when multiple opportunities overlap.

## Pending transfers

Do not include incoming transfer in available balance until destination settlement is complete.

Maintain explicit states:

```text
available
reserved
pending_out
in_transit
redeemable
settled
```

## Dynamic targets

Target allocation must be derived from current and measured conditions:

```text
actual allocated capital
frequency by trade direction
trade size distribution
executable liquidity
transfer latency
transfer cost
venue reliability
expected opportunity rate
```

If one direction dominates persistently, symmetric 50/50 can be inefficient.

The bot may eventually have different optimal allocation ratios for different venues and capital regimes.

## Shadow price

Scarce inventory has higher economic value.

Conceptual penalty:

```text
opportunity_economic_edge
=
market_net_edge
-
inventory_shadow_cost
```

A trade that restores scarce inventory may receive credit up to actual avoided rebalance liability.

This shadow cost must depend on actual remaining capacity, not a fixed nominal threshold.

## Capital-regime research

Research must explicitly test multiple capital states rather than optimize around one nominal amount.

Suggested scenario families are **examples, not product limits**:

```text
very small
small
medium
large
very large relative to pool depth
```

Use both:

```text
absolute amount buckets
+
relative fractions of usable inventory/liquidity
```

The purpose is to find transition points where:

```text
gas dominates
fee dominates
price impact dominates
inventory dominates
liquidity becomes the binding constraint
```

## Research tasks

- [ ] build balance/location/allocation schema
- [ ] define dedicated-wallet vs shared-wallet allocation semantics
- [ ] measure trade-size distribution from quote opportunities
- [ ] simulate multiple capital regimes rather than one 1,000-ALPH case
- [ ] simulate alternative base/quote allocations
- [ ] calculate base/quote capacity depletion dynamically
- [ ] estimate required native gas reserve as a function of expected actions
- [ ] design reservation semantics
- [ ] measure effect of pending transfer on available capacity
- [ ] derive target allocation from directional opportunity data
- [ ] derive `q_max` from balance + liquidity + risk constraints
- [ ] identify minimum viable capital per route from actual fee/gas economics
- [ ] identify capital level where additional funds stop increasing executable profit because liquidity becomes limiting

## Acceptance criteria

For any candidate trade and any operator-provided capital state, the model can answer:

```text
1. how much capital is actually spendable by the bot?
2. can both legs be funded now?
3. what is the maximum safe executable size now?
4. what q maximizes economic net profit?
5. what capacity remains after execution?
6. does this worsen a near-term shortage?
7. what future rebalance liability does it create?
8. should the correct decision simply be NO_TRADE?
```

The model must answer these without assuming a permanent nominal capital amount.
