# 02 — Asset Identity and Provenance

## Objective

Mencegah bug accounting/routing akibat menyamakan asset hanya berdasarkan symbol dan memastikan core dapat dipakai untuk economic asset serta chain lain.

**ALPH adalah first research profile, bukan hardcoded core asset.**

---

# 1. Core model

Pisahkan:

```text
EconomicAsset
```

dari:

```text
SettlementAsset
```

Generic examples:

```text
EconomicAsset = ASSET_X

SettlementAsset:
- native representation on ChainA
- wrapped representation on ChainB
- bridged representation on ChainC
- CEX custody balance
```

Current first-profile example:

```text
EconomicAsset = ALPH

SettlementAsset:
- ALPH_NATIVE_ALEPHIUM
- WALPH_ETHEREUM
- WALPH_BSC
- future ALPH_CEX_BALANCE
```

Likewise stable/value assets must remain location-specific settlement assets.

---

# 2. Asset identity must be runtime registry data

Core must not hardcode:

```text
ALPH token id
wALPH addresses
USDT addresses
decimals
bridge provenance
symbols
```

These belong to `AssetRegistry` records loaded/discovered and verified at runtime/startup.

Conceptual key:

```text
SettlementAssetId
=
(chain_or_location, native/token namespace, contract_or_asset_id)
```

not ticker text.

---

# 3. Asset registry schema

```text
EconomicAssetRecord {
    economic_asset_id
    display_symbol
    valuation_group
    metadata
    verification_state
}

SettlementAssetRecord {
    settlement_asset_id
    economic_asset_id
    chain_or_location

    native_or_token
    contract_or_asset_id
    decimals

    origin
    wrapper_or_bridge_provenance

    canonical_state
    source
    verification_state
    updated_at
}
```

---

# 4. Required rules

1. Symbol is display metadata, never primary identity.
2. Contract/asset ID + chain/location determines settlement identity.
3. Wrapped/bridged representations are not assumed operationally interchangeable until transfer/backing/provenance is verified.
4. Stablecoins with different provenance remain distinct settlement assets.
5. All authoritative raw amounts use integer base units.
6. No authoritative accounting in `f64`.
7. Decimal metadata is registry data, never a hidden per-token core constant.
8. Unknown asset representations fail closed until verified.
9. Asset aliases must resolve through explicit registry mapping.
10. Core economics consumes generic asset IDs and normalized valuation, not token-specific branches.

---

# 5. First-profile wALPH identities

Current research profile:

```text
Ethereum
0x590F820444fA3638e022776752c5eEF34E2F89A6

BNB Chain
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8
```

These are evidence rows for the ALPH profile, not values to embed in generic business logic.

---

# 6. Decimal / precision research

For every settlement asset, registry/research must capture:

```text
decimals
minimum transferable unit
bridge/transfer normalization precision
venue/router precision behavior
dust rules
rounding behavior
```

Runtime arithmetic uses raw integer quantities and explicit conversion metadata.

---

# 7. Transfer accounting fields

For transfer-capable assets:

```text
requested_raw
source_debited_raw
normalized_raw
dust_refunded_raw
destination_received_raw
```

Never assume:

```text
requested == received
```

without reconciliation.

---

# 8. Cross-chain portability requirement

Adding a new chain/asset representation should require:

```text
1. register/verify chain
2. register/verify settlement asset
3. map it to an EconomicAsset
4. register/discover markets
5. register transfer provenance if applicable
```

It must not require modifying profit formulas or inventory accounting code.

---

# 9. Research tasks

First ALPH profile:

- [ ] verify native ALPH decimals/raw unit
- [ ] verify Ethereum wALPH contract + decimals
- [ ] verify BSC wALPH contract + decimals
- [ ] verify Alephium bridged stablecoin asset IDs
- [ ] verify stablecoin contracts used by relevant EVM routes
- [ ] map bridge origin/provenance
- [ ] document normalization/dust behavior

Generic model:

- [ ] define stable canonical `EconomicAssetId`
- [ ] define stable canonical `SettlementAssetId`
- [ ] define chain/location namespace rules
- [ ] define asset registry validation lifecycle
- [ ] define runtime asset metadata refresh/versioning
- [ ] test a second unrelated economic asset profile without core changes

---

# Acceptance Criteria

Every quote, balance, transfer, and PnL record references an unambiguous settlement asset without relying on ticker text.

A second economic asset on a different chain set can be registered without changing core accounting/economic logic.
