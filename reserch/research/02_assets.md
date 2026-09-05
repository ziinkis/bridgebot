# 02 — Asset Identity and Provenance

## Objective

Mencegah bug accounting dan routing akibat menyamakan asset hanya berdasarkan symbol.

## Core model

Pisahkan:

```text
EconomicAsset
```

dari:

```text
SettlementAsset
```

Example:

```text
EconomicAsset = ALPH

SettlementAsset:
- ALPH_NATIVE_ALEPHIUM
- WALPH_ETHEREUM
- WALPH_BSC
- future ALPH_CEX_BALANCE
```

Likewise:

```text
EconomicAsset = USD_STABLE

SettlementAsset:
- USDT_ETHEREUM
- USDT_BSC
- USDTETH_ALEPHIUM
- USDTBSC_ALEPHIUM
- future USDT_CEX_LOCATION
```

## Baseline wALPH identities

```text
Ethereum
0x590F820444fA3638e022776752c5eEF34E2F89A6

BNB Chain
0x8683BA2F8b0f69b2105f26f488bADe1d3AB4dec8
```

These must still be independently re-verified before implementation.

## Asset registry schema

```text
SettlementAssetRecord {
    id
    economic_asset
    chain_or_location

    native_or_token
    contract_or_asset_id
    decimals

    bridge_origin
    bridge_destination_representation

    canonical
    source
    verification_state
}
```

## Required rules

1. Symbol is display metadata, never primary identity.
2. Contract/asset ID + chain/location determines settlement identity.
3. Wrapped/bridged representations are not assumed 1:1 operationally until bridge backing and route are verified.
4. Stablecoins with different bridge provenance remain distinct settlement assets.
5. All raw amounts use integer base units.
6. No authoritative accounting in `f64`.

## Decimal / precision research

For every asset, capture:

```text
decimals
minimum transferable unit
bridge normalization precision
DEX/router precision behavior
dust rules
rounding behavior
```

## Bridge accounting fields

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

## Research tasks

- [ ] verify native ALPH decimals/raw unit
- [ ] verify Ethereum wALPH contract + decimals
- [ ] verify BSC wALPH contract + decimals
- [ ] verify Alephium bridged stablecoin asset IDs
- [ ] verify Ethereum/BSC stablecoin contracts used by relevant routes
- [ ] map bridge origin/provenance
- [ ] document normalization/dust behavior
- [ ] define canonical internal IDs

## Acceptance criteria

Every quote, balance, transfer, and PnL record can reference an unambiguous settlement asset without relying on ticker text.
