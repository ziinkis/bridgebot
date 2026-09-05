# Open Questions

Research backlog untuk semua hal yang belum cukup terbukti.

## Market / venue

- [ ] Semua ALPH pools aktif di Alephium saat ini apa saja?
- [ ] Pool mana yang canonical vs spoof/duplicate/dead?
- [ ] Semua wALPH pools aktif di Ethereum apa saja?
- [ ] Semua wALPH pools aktif di BSC apa saja?
- [ ] Apakah ada route multi-hop yang secara konsisten mengalahkan direct route untuk size kecil?
- [ ] Venue mana yang punya quote/execution reliability tertinggi?

## Fees

- [ ] Fee aktual current-state untuk setiap ALPH pool Elexium?
- [ ] Fee tier ALPH routes di AYIN yang benar-benar executable?
- [ ] Apakah ada additional protocol/router fee selain LP fee?
- [ ] Bagaimana fee berubah ketika router membagi route?

## Liquidity / impact

- [ ] Berapa executable output untuk 5/10/20/40/60/75/100/125/150 ALPH di semua venue?
- [ ] Berapa size yang memaksimalkan absolute economic net profit per route?
- [ ] Berapa cepat executable depth berubah sepanjang hari?
- [ ] Apakah Alephium native side atau EVM side menjadi bottleneck utama?

## Slippage / latency

- [ ] Berapa quote drift p50/p95/p99 per venue/size?
- [ ] Detection-to-submit latency realistis berapa?
- [ ] Berapa edge yang biasanya bertahan cukup lama untuk dua-leg execution?
- [ ] Apakah private/protected Ethereum submission materially membantu?

## Gas

- [ ] Gas actual per route untuk current routers?
- [ ] Estimation error p95/p99?
- [ ] Native gas reserve minimum yang aman?
- [ ] Seberapa cepat Alephium UTXO fragmentation menjadi operational issue?

## Bridge

- [ ] Current exact supported asset/direction matrix?
- [ ] Current contract/configuration references?
- [ ] All bridge fees per direction?
- [ ] Actual cost untuk amount bucket kecil vs besar?
- [ ] Source-confirm → VAA-ready → redeem p50/p95/p99?
- [ ] Asset mana paling murah untuk rebalance: ALPH atau quote?
- [ ] Apa health signals paling cepat mendeteksi degraded bridge?
- [ ] Bagaimana bridge behaves during chain congestion / guardian delays?

## Inventory

- [ ] Apakah 50/50 initial allocation reasonable setelah flow data tersedia?
- [ ] Berapa p95 optimal trade size?
- [ ] Berapa minimum remaining trade capacity untuk mulai rebalance?
- [ ] Apakah target inventory harus asymmetric per venue?
- [ ] Seberapa sering natural reverse arbitrage menetralkan imbalance?

## Rebalancing

- [ ] Bagaimana mendefinisikan `R(state)` secara praktis dan computable?
- [ ] Kapan WAIT lebih murah daripada bridge?
- [ ] Kapan local swap lebih murah daripada transfer?
- [ ] Berapa batch bridge size yang economically efficient?
- [ ] Berapa hysteresis yang mencegah transfer oscillation?

## Execution risk

- [ ] Real probability leg A success / leg B failure?
- [ ] Expected unwind loss per route/size?
- [ ] Mana submission order terbaik per pair?
- [ ] Berapa max single-failure exposure yang acceptable untuk capital 1,000 ALPH-equivalent/venue?

## CEX future

- [ ] CEX mana yang benar-benar memiliki ALPH spot market + usable depth?
- [ ] Fee taker aktual?
- [ ] API/websocket reliability?
- [ ] Withdrawal network dan fee?
- [ ] Deposit/withdrawal maintenance frequency?
- [ ] Custody exposure limit per CEX?

## Economic viability

- [ ] Setelah all-in costs, berapa median and tail `ECONOMIC_NET_PNL`?
- [ ] Berapa opportunity frequency yang benar-benar executable?
- [ ] Berapa capital utilization per venue?
- [ ] Apakah expected return masih menarik setelah downtime, failed legs, and rebalancing?

## Rule

Jika pertanyaan belum dapat dijawab dari evidence yang direproduksi, statusnya tetap **UNKNOWN**. Jangan mengubahnya menjadi angka asumsi tanpa label eksplisit.
