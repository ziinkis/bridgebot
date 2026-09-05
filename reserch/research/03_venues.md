# 03 — Venue Model and Capabilities

## Objective

Mendefinisikan capability model yang cukup umum untuk DEX, CEX, dan future venue pada chain apa pun tanpa mengubah core arbitrage economics.

**ALPH venues adalah first research profile only.**

---

# 1. Core abstraction

```text
Venue
├── DEX
├── CEX
└── future execution venue types
```

Venue tidak identik dengan chain. Satu chain dapat mempunyai banyak venue; satu CEX dapat mempunyai banyak markets dan settlement networks.

Core arbitrage engine tidak boleh bergantung pada AMM tertentu, chain tertentu, atau symbol tertentu.

---

# 2. Generic identity model

Core references:

```text
ChainId
VenueId
MarketId
EconomicAssetId
SettlementAssetId
InventoryLocationId
```

Protocol/chain-specific names menjadi metadata registry, bukan core enum permanen.

Bad foundation:

```text
enum Chain {
    Alephium,
    Ethereum,
    Bsc
}
```

Preferred conceptual model:

```text
ChainRecord {
    id
    family
    network
    metadata
    adapter_kind
    capabilities
}
```

Chain baru ditambahkan sebagai data + adapter capability, bukan dengan rewrite opportunity evaluator.

---

# 3. Venue capability dimensions

```text
market_discovery
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
finality_model
```

Capabilities harus ditanyakan/dideklarasikan melalui registry, bukan diasumsikan sama untuk semua venues.

---

# 4. DEX-specific capabilities

Possible characteristics:

```text
AMM type
constant product
stable swap
concentrated liquidity
orderbook/on-chain book
multi-hop routing
split routing
fee tier / dynamic fee
block/state dependency
gas dependency
MEV exposure
same-chain atomicity semantics
```

Tidak semua DEX harus mengimplementasikan semua capability.

---

# 5. CEX-specific capabilities

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

CEX adapter menormalisasi semantics tersebut ke generic quote/execution state.

---

# 6. Chain adapter boundary

Venue adapter dan chain adapter harus terpisah ketika masuk akal.

Conceptual roles:

```text
ChainAdapter
    balance primitives
    transaction building/submission
    gas/fee estimation
    confirmation/finality
    contract/state reads
    native transfer

VenueAdapter
    market discovery
    quoting
    venue fee model
    route construction
    venue execution semantics
```

Contoh:

```text
EthereumChainAdapter
    reused by UniswapAdapter + other EVM venue adapters
```

atau future EVM-compatible chain dapat reuse sebagian besar chain-family adapter dengan network metadata yang berbeda.

---

# 7. No hardcoded deployment data

Venue/chain adapters tidak boleh bergantung pada hidden deployment constants untuk production universe.

Runtime deployment metadata berasal dari registry/config/discovery:

```text
factory addresses
router addresses
token addresses
pool ids
fee configuration
RPC endpoints
chain/network ids
```

Adapter code boleh memahami cara memakai data itu.

---

# 8. Suggested normalized interfaces

Conceptual only during research:

```text
ChainAdapter.capabilities(chain_id)
ChainAdapter.balance(location, asset)
ChainAdapter.estimate_fee(tx_plan)
ChainAdapter.submit(tx_plan)
ChainAdapter.status(handle)

VenueAdapter.discover_markets(venue_id)
VenueAdapter.quote(request) -> ExecutableQuote
VenueAdapter.simulate(plan) -> SimulationResult
VenueAdapter.execute(plan) -> ExecutionHandle
VenueAdapter.health() -> VenueHealth
```

Core tidak memanggil protocol-specific RPC methods secara langsung.

---

# 9. Venue state

```text
HEALTHY
DEGRADED
STALE
READ_ONLY
EXECUTION_DISABLED
UNKNOWN
```

Health berasal dari relevant signals, bukan satu ping.

---

# 10. Quote freshness

Staleness semantics berbeda per venue:

```text
DEX concentrated pool:
    state/block/tick/liquidity changed

CEX:
    local orderbook sequence no longer current

other chain:
    adapter-defined state reference
```

Quote object harus membawa state reference yang cukup untuk preflight/staleness validation.

---

# 11. Venue registry

```text
VenueRecord {
    venue_id
    venue_type
    chain_or_custody_location
    protocol_family
    protocol_version

    adapter_kind
    quote_method
    execution_method
    fee_model
    liquidity_model

    discovery_sources[]
    health_sources[]
    capabilities[]

    verification_state
    updated_at
    enabled
}
```

Tidak ada permanent `if venue == UNISWAP` di core evaluator.

---

# 12. Market registry

Markets harus dynamic records:

```text
MarketRecord {
    market_id
    venue_id
    settlement_assets[]
    pool_or_market_identifier
    fee_state
    route_metadata
    discovery_source
    verification_state
    last_observed_state
    execution_enabled
}
```

Market discovered tidak otomatis execution-enabled.

---

# 13. First-profile research tasks

- [ ] document Elexium quote/execution model
- [ ] document AYIN quote/router model
- [ ] document Nightshade executable behavior
- [ ] document Uniswap versions/routes used by current wALPH
- [ ] document PancakeSwap versions/routes used by current wALPH
- [ ] define venue-health minimum signals
- [ ] define quote freshness semantics

Generic architecture research:

- [ ] define ChainRegistry schema
- [ ] define VenueRegistry schema
- [ ] define MarketRegistry schema
- [ ] define adapter capability negotiation
- [ ] identify reusable chain-family adapters (e.g. EVM family)
- [ ] define validation lifecycle before execution enablement
- [ ] test whether a hypothetical second non-ALPH asset profile can be represented without core changes
- [ ] test whether a new chain can be represented without changing detector/evaluator/sizer

---

# Acceptance Criteria

A new venue can be added without changing economic logic of:

```text
buy at venue A
sell at venue B
```

A new chain can be added without changing:

```text
opportunity detection
economic evaluation
size optimization
inventory accounting
rebalance objective
```

Only registry data and required adapter implementations/capabilities should differ.
