# 11 — Rebalancing and Netting

## Objective

Menentukan cara termurah mengembalikan useful inventory capacity setelah arbitrage tanpa melakukan transfer berlebihan.

## Core principle

Rebalancing is not synonymous with bridging.

Priority order:

```text
1. NATURAL REVERSE ARBITRAGE
2. FLOW NETTING
3. WAIT WHILE SAFE
4. PARTIAL LOCAL REBALANCE
5. BATCH TRANSFER / BRIDGE
6. EMERGENCY REBALANCE
```

## Why netting matters

Example directional flow:

```text
+100 ALPH imbalance
-80 ALPH reverse imbalance
```

Net requirement:

```text
20 ALPH
```

Do not transfer 100 then 80 in the opposite direction.

## Rebalance liability

Define:

```text
R(state)
=
minimum expected economic cost to return inventory to acceptable region
```

Then a trade creates:

```text
DELTA_R = R(after_trade) - R(before_trade)
```

This is more accurate than assigning one full bridge fee to each trade.

## Min-cost flow model

Nodes:

```text
(location, settlement_asset)
```

Edges can include:

```text
DEX swap
bridge
native transfer
wait
natural expected reverse flow
future CEX deposit
future CEX withdrawal
```

Each edge carries:

```text
fee
fixed_cost
variable_cost
capacity
expected_latency
latency_distribution
risk
availability
```

Optimization objective:

```text
minimize total expected economic cost
```

subject to minimum venue trade capacity constraints.

## ALPH vs quote transfer

After `BUY ALPH on A / SELL ALPH on B`, both base and quote inventories move.

Planner must compare:

```text
move ALPH A → B
move quote B → A
local swaps
partial combination
wait for reverse opportunity
```

Select the path that restores the most useful capacity per unit economic cost.

## Depletion-aware trigger

A strong bridge/transfer trigger is based on lead time:

```text
time_to_floor
<=
rebalance_p95_latency + safety_margin
```

not simply `balance < 30%`.

## Hysteresis

Prevent oscillation.

Do not transfer at 49.9% and reverse transfer at 50.1%.

Use separate trigger and recovery targets.

Example concept only:

```text
trigger shortage at critical capacity
rebalance toward comfortable capacity
```

Exact bands must be derived from data.

## Economic vs emergency rebalance

### Economic

Batch is large enough that transfer cost is acceptable and inventory needs repair.

### Emergency

Inventory is projected to lose execution capacity before normal economic batch is reached.

Emergency may accept a higher cost if lost opportunity / failure risk is worse.

## Research tasks

- [ ] formalize R(state)
- [ ] simulate natural reverse-flow netting
- [ ] compare ALPH vs quote transfer cost
- [ ] model pending transfers
- [ ] model time-to-floor
- [ ] derive hysteresis policy
- [ ] compare economic vs emergency thresholds
- [ ] run min-cost-flow simulations

## Acceptance criteria

Given an inventory state, planner can rank `WAIT / REVERSE-FAVOR / LOCAL SWAP / BRIDGE BASE / BRIDGE QUOTE / PARTIAL MIX` by expected economic cost and risk.
