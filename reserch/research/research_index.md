# Research Index

Master index untuk seluruh research Bridgebot.

## Status legend

- `BASELINE` — sudah memiliki baseline reasoning tetapi masih perlu live validation.
- `IN PROGRESS` — sedang diisi dengan source/data aktual.
- `MEASURED` — memiliki dataset nyata yang dapat direproduksi.
- `VERIFIED` — fakta atau hasil telah diverifikasi silang.
- `BLOCKED` — membutuhkan dependency/data lain.
- `OPEN` — belum dikerjakan cukup dalam.

## Research map

| ID | File | Topic | Status | Primary output |
|---|---|---|---|---|
| V1 | `research_v1.md` | Baseline deep research | BASELINE | Initial system thesis |
| 01 | `01_market_map.md` | ALPH market/pool map | IN PROGRESS | Verified route universe + `data/pools/market_map_snapshot_2026-09-05.csv` |
| 02 | `02_assets.md` | Asset identity/provenance | OPEN | Settlement asset registry |
| 03 | `03_venues.md` | Venue capabilities | OPEN | Venue capability matrix |
| 04 | `04_dex_fees.md` | LP/trading fees | OPEN | Fee registry + evidence |
| 05 | `05_liquidity.md` | Executable liquidity | OPEN | Quote ladder dataset |
| 06 | `06_price_impact.md` | Size → impact curve | BLOCKED by 05 | Impact model |
| 07 | `07_slippage.md` | Quote drift / slippage | BLOCKED by live sampling | p50/p95/p99 drift |
| 08 | `08_gas.md` | Gas/execution cost | OPEN | Gas distribution |
| 09 | `09_bridge.md` | Bridge cost/latency/health | OPEN | Transfer economics |
| 10 | `10_inventory.md` | Inventory/capacity | BASELINE | Inventory state model |
| 11 | `11_rebalancing.md` | Netting/min-cost rebalance | BASELINE | Rebalance decision model |
| 12 | `12_execution_risk.md` | Failed leg/unwind/MEV | OPEN | Risk reserve model |
| 13 | `13_cex_extension.md` | Future DEX+CEX abstraction | BASELINE | Extensibility constraints |
| OQ | `open_questions.md` | Unknowns/blockers | OPEN | Research backlog |

## Priority order

```text
01_market_map
      ↓
02_assets
      ↓
03_venues
      ↓
04_dex_fees
      ↓
05_liquidity
      ↓
06_price_impact
      ↓
07_slippage
      ↓
08_gas
      ↓
09_bridge
      ↓
10_inventory
      ↓
11_rebalancing
      ↓
12_execution_risk
      ↓
13_cex_extension
```

Prioritas tidak berarti semua harus serial. Market map, assets, fees, gas, dan bridge research dapat berjalan paralel selama source/evidence disimpan dengan benar.

## Minimum dataset standard

Setiap measured row sebisa mungkin memiliki:

```text
timestamp_utc
chain
venue
pool_or_market
asset_in
asset_out
amount_in
amount_out
fee_tier
gas_estimate_or_actual
block_number_or_state_reference
source
notes
```

Bridge rows menambahkan:

```text
source_chain
destination_chain
requested_amount
actual_received
source_tx
vaa_or_message_reference
source_confirm_time
vaa_ready_time
redeem_time
settled_time
all_costs
```

## Decision gate

Tidak ada parameter production yang boleh dipindahkan ke `decisions/architecture_decisions.md` hanya karena terlihat masuk akal. Harus ada evidence atau alasan eksplisit mengapa parameter tersebut merupakan safety default sementara.
