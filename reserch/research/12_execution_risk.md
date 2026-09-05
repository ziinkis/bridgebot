# 12 — Execution Risk

## Objective

Mengukur risiko bahwa arbitrage yang terlihat profitable gagal karena execution tidak atomic, quote stale, partial completion, MEV/adverse selection, atau recovery path buruk.

## Core failure mode

Cross-chain / cross-venue arbitrage tidak memiliki satu atomic transaction yang menjamin kedua leg sukses.

Critical scenario:

```text
LEG A SUCCESS
LEG B FAIL
```

Hasilnya adalah directional exposure.

## Execution preparation

Preferred research flow:

```text
reserve both inventories
→ refresh quotes
→ validate quote freshness/state
→ simulate/preflight both legs where possible
→ precompute unwind
→ build/sign
→ near-concurrent submit
→ track independently
→ reconcile
→ hedge/unwind if needed
```

## Failed-leg reserve

Expected-value approximation:

```text
expected_failure_cost
=
P(failure) × expected_unwind_loss
```

Tetapi risk control juga membutuhkan hard limit:

```text
max_single_failure_loss <= allowed_risk_limit
```

Expected value saja tidak cukup untuk tail risk.

## Unwind research

Before executing an opportunity, identify:

```text
primary unwind route
secondary unwind route
maximum acceptable unwind loss
quote freshness for unwind
available inventory/gas for unwind
```

Do not discover the recovery plan only after one leg fails.

## Near-concurrent submission

Waiting full confirmation of leg A before submitting B may destroy edge.

But simultaneous submission creates independent failure paths.

Research must compare:

```text
submit ordering
latency
failure probability
expected exposure duration
```

per route pair.

## Quote freshness risk

Reject if:

```text
quote age > route-specific threshold
state/block mismatch invalidates quote
venue health degraded
expected remaining edge < required reserve
```

## Ethereum MEV / adverse selection

Research:

- public mempool exposure;
- protected/private transaction channels if applicable;
- effect on inclusion and revert behavior;
- whether protection materially reduces adverse fill for observed ALPH size.

Do not assume private submission is always superior; measure reliability and latency.

## CEX future risks

When CEX is added:

```text
partial fill
order reject
IOC/FOK semantics
orderbook sequence gap
API outage
withdrawal disabled
custodial balance lock
```

must enter the same risk framework.

## Route reliability metrics

Track:

```text
quote_success_rate
simulation_success_rate
submit_success_rate
revert_rate
fill_rate
median_inclusion
p95_inclusion
partial_completion_rate
unwind_frequency
unwind_loss_distribution
```

## Risk-adjusted threshold

Conceptual:

```text
dynamic_required_edge =
base_profit_floor
+
quote_drift_buffer
+
failed_leg_reserve
+
route_reliability_penalty
+
inventory_penalty
```

## Research tasks

- [ ] define execution state machine
- [ ] define reservation semantics
- [ ] measure quote-to-submit latency
- [ ] measure submit-to-confirm latency
- [ ] identify preflight/simulation support per venue
- [ ] design unwind quote capture
- [ ] estimate failed-leg loss by size bucket
- [ ] research Ethereum MEV protection options
- [ ] establish route reliability score

## Acceptance criteria

Every candidate opportunity has a quantified failure/recovery plan before it is considered execution-ready.
