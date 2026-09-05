# Quote Data

Folder untuk executable quote snapshots dan quote ladders.

Recommended fields:

```text
timestamp_utc
chain
venue
pool_or_route
asset_in
asset_out
amount_in
amount_out
effective_price
fee_tiers
estimated_gas
block_or_state
quote_method
success
error
```

Standard initial amount buckets:

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

Rules:

- capture bidirectional quotes where relevant;
- preserve failed quote attempts;
- retain state/block reference;
- do not convert one snapshot into a permanent liquidity claim;
- store enough data to reconstruct price-impact and net-profit curves.
