# 10 — Inventory Model

## Objective

Menentukan berapa inventory yang harus berada di setiap venue agar bot dapat mengeksekusi opportunity tanpa menunggu transfer, sambil menjaga capital efficiency di bawah constraint:

```text
MAX CAPITAL PER ACTIVE VENUE = 1,000 ALPH-equivalent
```

## Initial assumption

Untuk eksperimen pertama saja:

```text
~500 ALPH/wALPH
~500 quote-equivalent
```

Status: **ASSUMPTION**.

Target final harus berasal dari measured directional demand dan trade-size distribution.

## Inventory dimensions

Per venue/location:

```text
InventoryState {
    settlement_asset
    location

    available
    reserved
    locked
    pending_out
    pending_in
    in_transit
    redeemable

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

A venue with 1,000 ALPH-equivalent total but almost entirely base asset cannot keep buying; vice versa.

## Trade capacity metric

Ratio is useful but incomplete.

Define empirical capacity using expected trade size distribution.

Example:

```text
available base = 450 ALPH
p95 intended sell size = 75 ALPH

remaining sell capacity ≈ 6 p95 trades
```

Similar calculation for quote-side buy capacity.

## Initial safety zones

Ratio bands may be tested as secondary guard:

```text
35–65% base: healthy candidate band
25–35 / 65–75: warning candidate band
15–25 / 75–85: critical candidate band
<15 / >85: stop direction that worsens imbalance
```

All values are **ASSUMPTION**, pending simulation and measured flow.

## Inventory reservation

Before cross-venue execution:

```text
reserve both legs
```

Reserved balance cannot be used by another candidate.

This prevents overcommit when multiple opportunities overlap.

## Pending transfers

Do not include incoming transfer in available balance until destination settlement is complete.

## Dynamic targets

Long-term target should depend on:

```text
frequency by trade direction
trade size distribution
transfer latency
transfer cost
venue reliability
expected opportunity rate
```

If one direction dominates persistently, symmetric 50/50 can be inefficient.

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

## Research tasks

- [ ] build balance/location schema
- [ ] measure trade-size distribution from quote opportunities
- [ ] simulate 50/50 starting allocation
- [ ] simulate alternative allocations
- [ ] calculate base/quote capacity depletion
- [ ] estimate required native gas reserve
- [ ] design reservation semantics
- [ ] measure effect of pending transfer on available capacity
- [ ] derive target allocation from directional opportunity data

## Acceptance criteria

For any candidate trade, the model can answer:

```text
1. can both legs be funded now?
2. what capacity remains after execution?
3. does this worsen a near-term shortage?
4. what future rebalance liability does it create?
```
