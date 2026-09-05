# 08 — Gas and Execution Cost

## Objective

Mengukur actual transaction cost per chain/route dan membangun gas reserve yang cukup untuk normal execution + emergency recovery.

## Core rule

Jangan hardcode satu gas cost per chain.

Gas tergantung pada:

```text
route
router
hop count
contract path
state
transaction complexity
network gas price
```

## Alephium

Protocol minimum tidak sama dengan actual swap/contract execution cost.

Research flow:

```text
build
→ estimate
→ simulate/preflight
→ apply safety multiplier
→ submit
→ compare actual
```

Track UTXO-related complexity separately.

## Ethereum / BSC

For every candidate:

```text
estimated_gas_units
×
effective_gas_price
=
expected_native_cost
```

Convert expected native cost into common economic unit at the same decision timestamp.

## Gas observation schema

```text
GasObservation {
    timestamp
    chain
    venue
    route
    action_type
    amount_bucket

    estimated_units
    actual_units
    gas_price
    total_native_cost
    common_value_cost

    tx_hash
    block
    success
}
```

## Estimation error

Track:

```text
(actual - estimated) / estimated
```

Build p50/p95/p99 estimation error to choose a rational safety multiplier.

## Native gas reserve

Native gas balance is infrastructure inventory, not trading inventory.

Conceptual reserve target:

```text
p99 expected cost of:
normal swaps
+
emergency unwind
+
bridge/source actions
+
redeem actions
```

If gas balance drops below operational reserve, opening new risk may need to stop.

## Approval / setup transactions

Separate:

```text
one-time / amortized setup cost
```

from recurring trade cost.

Do not charge the full approval transaction against every trade.

## Alephium UTXO research

Track:

```text
UTXO count
small-output fragmentation
dust
build latency
input count
failure rate
```

Determine when idle consolidation is beneficial.

## Research tasks

- [ ] collect gas estimates for each route/size
- [ ] collect actual costs from test/real observed transactions where safe
- [ ] calculate estimation error distribution
- [ ] quantify native gas reserve requirement
- [ ] measure multi-hop vs direct route gas difference
- [ ] characterize Alephium UTXO fragmentation cost

## Output

Raw observations:

```text
reserch/data/gas/
```

## Acceptance criteria

Economic quote can subtract realistic route-specific gas cost and safety reserve with known confidence bounds.
