# Bridge Data

Folder untuk bridge transfer observations, cost breakdown, latency, VAA/message progression, dan reconciliation.

Recommended fields:

```text
timestamp_utc
source_chain
destination_chain
asset
requested_amount
source_debited_amount
normalized_amount
dust_refund
destination_received_amount
source_tx
source_confirm_time
finality_ready_time
vaa_or_message_reference
vaa_ready_time
redeem_tx
redeem_submit_time
redeem_confirm_time
settled_time
source_gas_cost
destination_gas_cost
protocol_or_message_fee
relayer_fee
other_cost
common_value_total_cost
status
error
```

Derived metrics:

```text
source_to_confirm_latency
confirm_to_vaa_latency
vaa_to_redeem_latency
total_settlement_latency
bridge_cost_bps
p50/p90/p95/p99 latency
```

Rules:

- in-transit amount is never counted as available inventory;
- store failed/stuck/retried transfers;
- distinguish source submitted, VAA ready, redeemed, and reconciled;
- use actual received amount for accounting;
- bridge health research should use recent real lifecycle data, not UI availability alone.
