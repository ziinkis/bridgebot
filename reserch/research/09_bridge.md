# 09 — Alephium Bridge Research

## Objective

Mengukur bridge sebagai **settlement/rebalancing system**, bukan menganggapnya sebagai instant swap path.

Research harus menjawab:

```text
what assets can move?
which directions are supported?
what exact contracts/components are involved?
what are all costs?
what is actual latency distribution?
what does healthy/degraded/paused mean?
when is bridging economically justified?
```

## Baseline thesis

Normal cross-chain arbitrage should use prefunded inventory.

Bridge default role:

```text
inventory rebalance
```

not:

```text
trade hot-path dependency
```

## Transfer lifecycle

Research state model:

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

Failure states:

```text
BRIDGE_PAUSED
SOURCE_FAILED
SOURCE_REORG
VAA_TIMEOUT
REDEEM_FAILED
BALANCE_MISMATCH
ACCOUNTING_MISMATCH
```

## Bridge cost decomposition

Never use one guessed percentage.

Track:

```text
source approval/setup amortization
source transaction gas
message/protocol fee
gas on destination redeem
relayer fee if used
rounding/dust
capital-in-transit opportunity cost
operational/risk premium
```

## Cost normalization

```text
bridge_cost_bps =
all_bridge_cost_in_common_value
/ transferred_value
× 10,000
```

This exposes why tiny transfers can be uneconomic when fixed costs dominate.

## Economic minimum batch

Conceptual formula:

```text
Q_MIN =
FIXED_TRANSFER_COST × 10,000
/ MAX_ACCEPTABLE_TRANSFER_BPS
```

Values are live/measured parameters, not constants in this document.

## Latency measurement

For each transfer direction, record:

```text
source_submit_time
source_confirm_time
finality_ready_time
vaa_ready_time
redeem_submit_time
redeem_confirm_time
settled_time
```

Calculate:

```text
p50
p90
p95
p99
```

Never use a single documentation estimate as runtime truth.

## Bridge trigger thesis

Preferred trigger considers depletion lead time:

```text
if time_to_inventory_floor
<=
bridge_p95_latency + safety_margin
then rebalance urgency rises
```

But bridge should still compete against:

```text
reverse arbitrage
flow netting
wait
local swap
moving quote asset instead of ALPH
partial rebalance
```

## Asset choice

After an arbitrage, both base and quote inventory drift.

Therefore the bridge planner must compare moving:

```text
ALPH/wALPH
quote assets such as supported stablecoins
```

whichever restores useful trade capacity at lower total economic cost.

## Pending balance rule

```text
in_transit != available
```

Track separately:

```text
available
reserved
pending_out
in_transit
redeemable
settled
```

## Health model

Do not define bridge health using one API response.

Research signals:

```text
source chain progression
destination chain progression
contract paused/config state
recent successful source transfers
VAA progression
recent VAA latency
redeem simulation
stuck/backlog rate
```

## Incident-aware research

Because the bridge has had a recent security incident and relaunch, research must preserve a stricter threat model around off-chain observation, VAA progression, contract state, and operational health. Do not infer safety solely from UI availability.

## Research tasks

- [ ] verify current supported asset/direction map
- [ ] verify current bridge contracts/configuration
- [ ] document VAA/message lifecycle
- [ ] measure source + destination fees
- [ ] measure actual transfer cost by amount bucket
- [ ] measure latency distribution per direction
- [ ] test/research dust/normalization behavior
- [ ] define health state inputs
- [ ] define timeout/recovery semantics
- [ ] compare ALPH vs stablecoin rebalance paths

## Output

Raw observations:

```text
reserch/data/bridge/
```

## Acceptance criteria

Given a real inventory imbalance, research model can estimate the cost, latency, risk, and expected recovered trade capacity for each possible bridge path.
