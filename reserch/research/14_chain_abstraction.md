# 14 — Chain Abstraction and Runtime Registry

## Objective

Membuktikan bahwa Bridgebot dapat memakai chain baru tanpa rewrite core arbitrage economics.

ALPH/Alephium/Ethereum/BSC hanyalah first validation profile.

---

# 1. Required property

Core components berikut harus tetap unchanged ketika chain baru ditambahkan:

```text
opportunity detection
economic evaluation
size optimization
inventory accounting
risk budgeting
rebalance objective
PnL accounting
```

Yang boleh berubah/ditambah:

```text
chain registry data
asset registry data
venue registry data
market discovery data
chain adapter
venue adapter
transfer adapter
operator policy
```

---

# 2. Chain families

Chain abstraction harus memungkinkan reuse berdasarkan family/capability, misalnya:

```text
EVM-like account chain
UTXO chain
Solana-like account/program chain
Cosmos-like chain
future/custom chain
```

Jangan membuat assumption bahwa semua chain mempunyai:

```text
EVM address
ERC-20 decimals API
eth_estimateGas
mempool semantics
single gas token behavior
same finality model
```

Core menggunakan capabilities, adapter yang menerjemahkan semantics.

---

# 3. Chain registry schema

Conceptual:

```text
ChainRecord {
    chain_id
    family
    network_id
    display_name

    native_gas_asset
    adapter_kind

    rpc_endpoints[]
    websocket_endpoints[]
    explorer_metadata

    finality_model
    confirmation_policy

    capabilities[]

    source
    verification_state
    updated_at
    enabled
}
```

Tidak ada requirement bahwa `chain_id` adalah EVM integer.

---

# 4. Chain capabilities

Possible capability flags:

```text
ACCOUNT_BALANCE
TOKEN_BALANCE
UTXO_BALANCE
CONTRACT_CALL
SIMULATION
GAS_ESTIMATION
FEE_MARKET
MEMPOOL
WEBSOCKET_BLOCKS
TOKEN_METADATA
NATIVE_TRANSFER
MULTICALL
FINALITY_QUERY
TRANSACTION_REPLACEMENT
```

Core/venue adapters negotiate required capabilities.

---

# 5. Runtime configuration precedence

Proposed source precedence must be researched:

```text
operator policy
        ↓
validated local/remote registry
        ↓
authoritative protocol metadata
        ↓
on-chain discovery/state
        ↓
measured runtime health
```

Conflicts must fail closed unless a deterministic policy resolves them.

---

# 6. No hidden constants

Research must audit future implementation for hidden chain-specific constants such as:

```text
addresses
decimals
fee tiers
confirmation counts
RPC URLs
router ids
bridge ids
pool ids
market symbols
slippage
profit threshold
size limits
```

Protocol behavior may be code; deployment/state/config data must be externalized or discovered.

---

# 7. Registry validation

Dynamic data lifecycle:

```text
DISCOVERED
→ SOURCE_VERIFIED
→ IDENTITY_VERIFIED
→ CAPABILITY_VERIFIED
→ HEALTH_CHECKED
→ ENABLED
```

A changed registry record should invalidate dependent quote/execution plans when relevant.

---

# 8. Adapter lifecycle

Conceptual:

```text
load ChainRecord
        ↓
resolve adapter_kind
        ↓
validate required capabilities
        ↓
initialize endpoints/state readers
        ↓
health check
        ↓
serve normalized primitives
```

Core should not know how those primitives are implemented.

---

# 9. Portability proof

Before implementation architecture is accepted, perform at least one paper/prototype portability test:

```text
PROFILE A
ALPH across Alephium/Ethereum/BSC

PROFILE B
unrelated EconomicAsset on a different chain combination
```

Success condition:

```text
no changes to core opportunity/economics/inventory/rebalance formulas
```

Only registry records and adapters differ.

---

# 10. Failure modes to research

- incompatible decimal/amount models
- chains without conventional mempool
- probabilistic vs deterministic finality
- RPC endpoint disagreement
- dynamic fee market differences
- chain reorg semantics
- token metadata spoofing
- registry compromise/staleness
- protocol upgrades changing adapter behavior
- multiple native gas assets or sponsored transactions
- chains with unusual account/UTXO locking semantics

---

# Research Tasks

- [ ] define stable generic `ChainId`
- [ ] define ChainRegistry schema
- [ ] define chain capability model
- [ ] define adapter interface boundary
- [ ] classify Alephium vs EVM differences
- [ ] identify what EVM adapter logic can be reused across Ethereum/BSC
- [ ] define endpoint health/failover semantics
- [ ] define registry source/version/signature/trust model
- [ ] define hot reload/change invalidation semantics
- [ ] audit current research files for accidental ALPH/Alephium core assumptions
- [ ] design second-profile portability test
- [ ] prove new chain addition does not require detector/evaluator/sizer rewrite

---

# Acceptance Criteria

`VERIFIED` only when a second chain/asset profile can be represented with:

```text
new registry data
+ adapter(s) when semantics are new
```

while these remain unchanged:

```text
opportunity model
profit model
size optimizer
inventory state model
rebalance optimization objective
PnL layers
```

If core contains chain-specific branches that are not generic capability handling, research must classify them as architecture debt before implementation starts.
