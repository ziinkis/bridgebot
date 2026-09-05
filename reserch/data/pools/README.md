# Pool / Market Data

Folder untuk raw dan normalized market-map observations.

Recommended fields:

```text
timestamp_utc
chain
venue
venue_type
pool_or_market
factory_or_parent
pool_type
asset0
asset1
economic_asset0
economic_asset1
decimals0
decimals1
fee_tier
router
block_or_state
status
source
```

Rules:

- preserve raw source where possible;
- every snapshot must be timestamped;
- do not overwrite historical observations when pool config changes;
- canonical/spoof/unknown status must be explicit;
- UI appearance alone is not verification.
