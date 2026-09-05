# Gas Data

Folder untuk gas estimates, actual transaction costs, dan estimation-error observations.

Recommended fields:

```text
timestamp_utc
chain
venue
route
action_type
amount_bucket
estimated_gas_units
actual_gas_units
gas_price
estimated_native_cost
actual_native_cost
common_value_cost
tx_hash
block
success
```

Derived metrics:

```text
estimation_error_pct
p50/p95/p99 gas units
p50/p95/p99 native cost
route-specific gas distribution
```

Rules:

- distinguish estimate from actual;
- distinguish recurring trade cost from setup/approval cost;
- retain failed/reverted transactions where useful;
- native gas reserve research should derive from distributions, not one sample.
