# 13 — CEX Extension

## Objective

Memastikan research/architecture awal tidak mengunci sistem hanya untuk DEX. CEX nanti harus dapat ditambahkan tanpa rewrite core economic model.

## Core abstraction

Do not define core as:

```text
DEX arbitrage engine
```

Prefer:

```text
Venue arbitrage engine

Venue
├── DEX
└── CEX
```

Likewise:

```text
Transfer
├── bridge
├── native chain transfer
├── CEX deposit
└── CEX withdrawal
```

## Executable quote normalization

DEX quote:

```text
amount_in
→ AMM/router
→ expected amount_out
```

CEX quote must walk orderbook depth.

Do not use only best bid/ask for requested size.

Example:

```text
asks:
100 @ P1
200 @ P2
500 @ P3
```

A 250 ALPH buy consumes multiple levels.

Normalized result should still become `ExecutableQuote`.

## CEX costs

Potential recurring costs/risk:

```text
maker fee
taker fee
orderbook impact
partial fill risk
API latency
order rejection
withdrawal fee
deposit/withdrawal latency
network availability
custody/location risk
```

## Initial execution preference

For future hot-path arbitrage, research should first evaluate taker / protected IOC-style execution rather than maker strategies because certainty matters more than nominal fee optimization.

Maker strategies are a separate research problem.

## Partial fills

Critical state:

```text
requested buy = 100 ALPH
CEX fill = 37
other venue leg = 100
```

Execution coordinator must use actual filled quantity as authoritative and maintain hedge/recovery logic.

## Order states

Future normalized states must support at least:

```text
NEW
OPEN
PARTIAL
FILLED
CANCEL_PENDING
CANCELED
REJECTED
EXPIRED
```

## Market data integrity

Preferred CEX orderbook model:

```text
REST snapshot
+
WebSocket ordered deltas
→ local book
```

Sequence gaps invalidate the local book until resync.

Do not trade on an orderbook whose update sequence is uncertain.

## CEX inventory

CEX balance is a distinct custody location:

```text
MEXC_SPOT
BINANCE_SPOT
GATE_SPOT
...
```

Track:

```text
available
locked
reserved
pending withdrawal
pending deposit
```

Do not treat CEX ALPH as identical settlement location to wallet ALPH.

## Withdrawal/deposit as rebalance

Normal model remains prefunded:

```text
BUY CEX + SELL DEX
```

without waiting for withdrawal.

Then later:

```text
net flows
→ choose withdrawal/deposit only if economically required
```

## Transfer metadata

Future CEX transfer edge requires:

```text
asset
network
min withdrawal
max withdrawal
withdrawal fee
withdrawal status
deposit status
confirmations
expected latency
address/tag requirements
limits
```

## Research tasks before adding first CEX

- [ ] identify CEX with real ALPH markets and sufficient depth
- [ ] verify official API/websocket capability
- [ ] verify fee schedule
- [ ] measure orderbook depth by standard size bucket
- [ ] measure websocket/update latency
- [ ] verify IOC/FOK/order semantics
- [ ] map deposit/withdrawal networks
- [ ] measure withdrawal fees and latency
- [ ] define custody risk constraints

## Acceptance criteria

Adding a CEX requires venue/transfer adapters and CEX-specific risk logic, but should not require redesigning core opportunity economics, inventory ledger, or rebalance graph.
