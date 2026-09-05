# 06 — Price Impact

## Objective

Mengukur deterioration yang disebabkan oleh trade size kita sendiri terhadap liquidity curve.

## Definition

```text
PRICE IMPACT
=
perbedaan antara reference/marginal price dan effective execution price
karena size trade pada liquidity state yang sama
```

Ini berbeda dari slippage/quote drift yang terjadi karena state berubah setelah quote.

## Why it matters

Untuk modal 1,000 ALPH-equivalent/venue, trade terlalu besar dapat membuat apparent spread terlihat profitable tetapi absolute net profit justru turun karena curve impact.

Example shape:

```text
size 10   -> high edge, low absolute profit
size 75   -> lower edge, highest absolute profit
size 150  -> edge collapses or turns negative
```

Maka optimizer mencari:

```text
argmax economic_net(size)
```

bukan maximum possible size.

## Measurement

Price impact research tergantung pada quote ladder dari `05_liquidity.md`.

For each route/state:

```text
size
reference_price
effective_price
impact_bps
amount_out
```

## Important AMM note

Untuk concentrated liquidity, jangan menyimpulkan impact dari total reserve/TVL sederhana. Gunakan protocol quoter/simulation/state-aware math yang sesuai.

## Multi-hop

Impact route adalah hasil gabungan semua hops dan tidak boleh diasumsikan sebagai sum sederhana tanpa menghitung actual executable output.

## Required outputs

- impact curve per route
- marginal output deterioration
- size at maximum absolute net profit
- size where net edge hits minimum allowed threshold
- size where route becomes economically unusable

## Research tasks

- [ ] derive impact from quote ladders
- [ ] compare direct vs multi-hop impact
- [ ] compare fee-tier/liquidity trade-off
- [ ] repeat measurements across time/state
- [ ] identify liquidity regime changes

## Acceptance criteria

Untuk setiap major route, tersedia empirical curve yang cukup untuk menjawab:

```text
berapa additional ALPH masih profitable untuk ditambahkan ke trade ini?
```
