# 04 — DEX Fees

## Objective

Membangun **fee registry yang evidence-based** untuk setiap pool/route yang benar-benar dapat dieksekusi.

Fee tidak boleh disimpan sebagai satu konstanta per DEX.

Bad:

```text
UNISWAP_FEE = 1%
AYIN_FEE = 0.3%
```

Preferred:

```text
fee belongs to the exact pool/route/state/configuration
```

## Baseline observations to re-verify

Research v1 menemukan baseline berikut:

```text
Uniswap V3 ALPH pools observed: 1.00%
PancakeSwap V3 ALPH pool paths observed: 1.00%
Elexium documented standard: ~0.30%, configurable
AYIN: classic/concentrated pools with multiple possible fee tiers
```

Status semua angka di atas tetap `BASELINE` sampai current pool state diverifikasi dan disimpan di dataset.

## Multiplicative fee math

Dua leg fee tidak sekadar dijumlahkan jika ingin precise retained-value ratio.

Example:

```text
fee A = 0.30%
fee B = 1.00%

retained = 0.997 × 0.99
fee_floor = 1 - retained
          = 1.297%
          = 129.7 bps
```

## Important accounting rule

Jika executable quoter sudah menghasilkan `amount_out` setelah pool fee + deterministic AMM impact, jangan kurangi fee dan impact lagi dari `amount_out` saat menghitung final PnL.

Fee tetap disimpan sebagai telemetry agar dapat menjelaskan economics dan membuat coarse prefilter.

## Route fee record

```text
FeeObservation {
    timestamp
    chain
    venue
    pool
    route
    fee_tier
    fee_source
    block_or_state
    verified
}
```

For multi-hop:

```text
route = [pool_a, pool_b, ...]
fees  = [fee_a, fee_b, ...]
```

## Coarse prefilter

Pool-fee floor dapat digunakan untuk menghindari quote mahal jika apparent spread jelas tidak cukup.

Tetapi:

```text
coarse fee filter != execution threshold
```

Final decision selalu menggunakan executable quote + actual cost model.

## Research tasks

- [ ] verify fee config Elexium ALPH pools
- [ ] enumerate AYIN fee tiers for ALPH routes
- [ ] verify Uniswap ALPH pool fee tiers
- [ ] verify Pancake ALPH pool fee tiers
- [ ] detect routes whose effective fee changes through routing
- [ ] identify protocol/router fees beyond LP fee if any
- [ ] timestamp every fee observation

## Acceptance criteria

Untuk setiap execution-eligible route, bot research dataset dapat menjawab `what exact trading fees are embedded in this quote and why?`.
