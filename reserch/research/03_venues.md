# 03 — Venue Model and Capabilities

## Objective

Mendefinisikan capability model yang cukup umum untuk DEX sekarang dan CEX nanti.

Core abstraction:

```text
Venue
├── DEX
└── CEX
```

Arbitrage engine tidak boleh bergantung langsung pada konsep AMM tertentu.

## Venue capability dimensions

```text
market_data
executable_quote
simulation_or_preflight
execution
balance_query
order_or_tx_tracking
fee_query
route_discovery
transfer_support
health_status
```

## DEX-specific capabilities

Possible characteristics:

```text
AMM type
concentrated liquidity
multi-hop routing
router split routing
fee tier
block/state dependency
gas dependency
MEV exposure
atomic transaction semantics within one chain
```

## Future CEX-specific capabilities

```text
orderbook websocket
snapshot + delta sequencing
maker/taker fees
IOC/FOK support
partial fill behavior
account balances
locked balances
withdrawal/deposit status
network-specific fees
custodial risk
```

## Suggested normalized interfaces

Conceptual only during research:

```text
Venue.quote(request) -> ExecutableQuote
Venue.balance(asset) -> VenueBalance
Venue.health() -> VenueHealth
Venue.execute(order) -> ExecutionHandle
Venue.status(handle) -> ExecutionStatus
```

This does not imply implementation language/API yet.

## Venue state

Track:

```text
HEALTHY
DEGRADED
STALE
READ_ONLY
EXECUTION_DISABLED
UNKNOWN
```

Health must be derived from relevant signals, not one ping.

## Quote freshness

Each venue needs a model for when a quote becomes unusable.

Examples:

```text
DEX: block/state changed
CEX: orderbook sequence advanced materially
```

Quote object must carry a state reference sufficient to detect staleness.

## Venue registry record

```text
VenueRecord {
    venue_id
    type
    chain_or_custody_location
    protocol_version

    quote_method
    execution_method
    fee_model
    liquidity_model

    health_sources[]
    capabilities[]

    source
    status
}
```

## Research tasks

- [ ] document Elexium quote/execution model
- [ ] document AYIN quote/router model
- [ ] document Nightshade executable behavior
- [ ] document Uniswap version/routes used by wALPH
- [ ] document PancakeSwap version/routes used by wALPH
- [ ] define venue-health minimum signals
- [ ] define quote freshness semantics per venue
- [ ] reserve extension points for CEX orderbooks

## Acceptance criteria

A new venue can be added later without changing the economic logic of `buy venue A / sell venue B`; only venue-specific adapter/capability behavior should differ.
