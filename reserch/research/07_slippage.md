# 07 — Slippage / Quote Drift

## Objective

Mengukur adverse movement antara quote dan actual/near-future executable state untuk menentukan slippage guard yang berbasis data.

## Definition

```text
PRICE IMPACT
= deterioration karena trade size pada state yang sama

SLIPPAGE / QUOTE DRIFT
= deterioration karena state berubah setelah quote dibuat
```

## Why fixed slippage is dangerous

Untuk arbitrage, `0.5%` atau `1%` bisa lebih besar daripada net profit target.

A transaction can succeed while the arbitrage is already economically negative.

Preferred concept:

```text
max_allowed_deterioration
<=
expected_net_edge
- required_final_profit
- execution_risk_reserve
```

## Measurement protocol

Untuk setiap detected quote, simpan snapshots:

```text
t0
+250ms if available
+500ms
+1s
+2s
next relevant state/block
expected inclusion window
```

Tidak semua chain/venue membutuhkan interval sama; protocol harus disesuaikan terhadap realistic inclusion/update cadence.

## Distribution

Per:

```text
venue
route
side
size bucket
market regime
```

calculate:

```text
p50 adverse drift
p90
p95
p99
worst observed
```

Positive/favorable movement harus dipisahkan dari adverse tail.

## Dynamic slippage model

Initial target model:

```text
slippage_guard =
min(
    profit_budget_remaining,
    empirically_required_adverse_drift_buffer
)
```

Never set tolerance larger than what preserves required final economics.

## Stale quote rejection

Research harus menentukan per venue:

```text
max quote age
state mismatch behavior
block/sequence invalidation
```

## CEX extension

Untuk CEX, equivalent slippage research adalah orderbook movement/fill deterioration antara decision dan fill, termasuk partial fills.

## Research tasks

- [ ] build quote time-series collector
- [ ] collect adverse drift per venue
- [ ] bucket by size
- [ ] bucket by volatility/regime
- [ ] determine p95/p99 guard candidates
- [ ] compare detection-to-submit latency with quote decay
- [ ] identify stale-state rejection rules

## Acceptance criteria

Slippage tolerance dapat dijelaskan dari measured adverse-drift distribution dan profit budget, bukan angka arbitrer.
