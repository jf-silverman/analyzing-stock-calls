# Ticker corrections log

GENERATED — rebuilt from the `ticker_reviews` table on every pipeline run. Do not edit by hand; changes are overwritten. This records ticker↔company corrections so one company no longer fans out across bogus symbols: the ingest-time guard (`reconcile_ingest_tickers` in `code/pipeline.py`) going forward, plus the one-time 2026-10-20 backlog cleanup that prompted it.

- **Remapped** — the model's symbol wasn't a real ticker; auto-pointed to the real keeper.
- **Held (no match)** — not a real ticker and no confident match found; stored as `????` for review.
- **Flagged (wrong company)** — the symbol is a *different* real company; held as `????` (never auto-remapped, since the company name could be the wrong half).

_80 correction(s) recorded._

| Episode | Model said | Company | Action | Result | Reported? |
|---------|-----------|---------|--------|--------|-----------|
| 2026-10-30 | `IBM` | IBM | Flagged (wrong company) | is International Business Machines | yes |
| 2026-10-22 | `BRK.B` | Berkshire Hathaway | Remapped | → `BRK-B` | yes |
| 2026-10-22 | `FL` | Foot Locker | Held (no match) | held `????` | yes |
| 2026-09-09 | `FXFR` | FedEx Freight | Remapped | → `FDXF` | yes |
| 2026-09-08 | `HNGR` | Hinge Health | Remapped | → `HNGE` | yes |
| 2026-09-08 | `MK` | Merck | Remapped | → `MRK` | yes |
| 2026-09-08 | `VERNOVA` | GE Vernova | Remapped | → `GEV` | yes |
| 2026-09-03 | `FBLK` | Five Below | Remapped | → `FIVE` | yes |
| 2026-08-31 | `AUY` | Agnico Eagle Mines | Remapped | → `AEM` | yes |
| 2026-08-31 | `FXFR` | FedEx Freight | Remapped | → `FDXF` | yes |
| 2026-08-18 | `FIGMA` | Figma | Remapped | → `FIG` | yes |
| 2026-08-13 | `CRVG` | CoreWeave | Remapped | → `CRWV` | yes |
| 2026-07-30 | `ESTL` | Establishment Labs | Remapped | → `ESTA` | yes |
| 2026-07-16 | `DGE` | Diageo | Remapped | → `DEO` | yes |
| 2026-07-16 | `QNTA` | Quanta Services | Remapped | → `PWR` | yes |
| 2026-07-09 | `HXSCL` | SK Hynix | Remapped | → `SKHY` | yes |
| 2026-07-09 | `LMS` | Lemonade | Remapped | → `LMND` | yes |
| 2026-07-08 | `FDEX` | FedEx Freight | Remapped | → `FDXF` | yes |
| 2026-07-06 | `SLST` | Solstice Advanced Materials | Remapped | → `SOLS` | yes |
| 2026-06-29 | `ONEOK` | ONEOK | Remapped | → `OKE` | yes |
| 2026-06-18 | `FDRT` | FedEx Freight | Remapped | → `FDXF` | yes |
| 2026-06-10 | `DKGS` | DraftKings | Remapped | → `DKNG` | yes |
| 2026-06-05 | `FXFR` | FedEx Freight | Remapped | → `FDXF` | yes |
| 2026-06-05 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2026-05-29 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2026-05-28 | `HNGR` | Hinge Health | Remapped | → `HNGE` | yes |
| 2026-05-26 | `005930` | Samsung Electronics | Remapped | → `SSNLF` | yes |
| 2026-05-26 | `HXSCL` | SK Hynix | Remapped | → `SKHY` | yes |
| 2026-05-15 | `BWC` | Babcock & Wilcox | Remapped | → `BW` | yes |
| 2026-05-11 | `APOE` | Applied Optoelectronics | Remapped | → `AAOI` | yes |
| 2026-05-11 | `VCOR` | Vicor | Remapped | → `VICR` | yes |
| 2026-05-08 | `MICRON` | Micron Technology | Remapped | → `MU` | yes |
| 2026-05-07 | `BEE` | Bloom Energy | Remapped | → `BE` | yes |
| 2026-05-06 | `PNTS` | Palantir Technologies | Remapped | → `PLTR` | yes |
| 2026-05-06 | `SOLST` | Solstice Advanced Materials | Remapped | → `SOLS` | yes |
| 2026-05-05 | `CWVU` | CoreWeave | Remapped | → `CRWV` | yes |
| 2026-05-01 | `ASTR` | Astera Labs | Remapped | → `ALAB` | yes |
| 2026-05-01 | `ONSEMI` | ON Semiconductor | Remapped | → `ON` | yes |
| 2026-04-27 | `TLNE` | Talen Energy | Remapped | → `TLN` | yes |
| 2026-04-22 | `ASRA` | Astera Labs | Remapped | → `ALAB` | yes |
| 2026-04-20 | `NOVA` | Enphase Energy | Remapped | → `ENPH` | yes |
| 2026-04-20 | `SNCR` | Synchrony Financial | Remapped | → `SYF` | yes |
| 2026-04-17 | `ALPHABET` | Alphabet | Remapped | → `GOOG` | yes |
| 2026-04-15 | `BCFM` | Broadcom | Remapped | → `AVGO` | yes |
| 2026-04-10 | `SNE` | Sony | Remapped | → `SONY` | yes |
| 2026-04-09 | `DWVG` | D-Wave Systems | Remapped | → `QBTS` | yes |
| 2026-04-09 | `RDDM` | Reddit | Remapped | → `RDDT` | yes |
| 2026-04-06 | `NVNT` | Navient | Remapped | → `NAVI` | yes |
| 2026-04-01 | `CORE` | CoreWeave | Remapped | → `CRWV` | yes |
| 2026-03-27 | `CORE` | CoreWeave | Remapped | → `CRWV` | yes |
| 2026-03-24 | `EXXN` | Exxon Mobil | Remapped | → `XOM` | yes |
| 2026-03-10 | `CREDO` | Credo Technology Group | Remapped | → `CRDO` | yes |
| 2026-03-10 | `SEAG` | Seagate Technology | Remapped | → `STX` | yes |
| 2026-03-09 | `BROADCOM` | Broadcom | Remapped | → `AVGO` | yes |
| 2026-03-09 | `LMTM` | Lumentum | Remapped | → `LITE` | yes |
| 2026-03-09 | `LWAN` | Lamb Weston | Remapped | → `LW` | yes |
| 2026-03-06 | `CSII` | Casey's General Stores | Remapped | → `CASY` | yes |
| 2026-03-05 | `BROADCOM` | Broadcom | Remapped | → `AVGO` | yes |
| 2026-03-05 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2026-03-02 | `ARESZ` | Ares Management | Remapped | → `ARES` | yes |
| 2026-02-06 | `VRTV` | Vertiv | Remapped | → `VRT` | yes |
| 2026-02-04 | `LILY` | Eli Lilly | Remapped | → `LLY` | yes |
| 2026-02-02 | `TTIC` | Tyler Technologies | Remapped | → `TYL` | yes |
| 2026-01-29 | `RTZ` | Rio Tinto | Remapped | → `RIO` | yes |
| 2026-01-27 | `CREDO` | Credo Technology Group | Remapped | → `CRDO` | yes |
| 2026-01-21 | `FLUTTER` | Flutter Entertainment | Remapped | → `FLUT` | yes |
| 2026-01-16 | `ABV` | AbbVie | Remapped | → `ABBV` | yes |
| 2026-01-05 | `FI` | Fiserv | Remapped | → `FISV` | yes |
| 2025-12-18 | `CREDO` | Credo Technology Group | Remapped | → `CRDO` | yes |
| 2025-12-17 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2025-12-11 | `NCL` | Norwegian Cruise Line | Remapped | → `NCLH` | yes |
| 2025-12-10 | `RQTI` | Rigetti Computing | Remapped | → `RGTI` | yes |
| 2025-12-01 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2025-12-01 | `THUP` | ThredUP | Remapped | → `TDUP` | yes |
| 2025-11-24 | `SIX` | Six Flags Entertainment | Remapped | → `FUN` | yes |
| 2025-11-20 | `GPS` | Gap Inc. | Remapped | → `GAP` | yes |
| 2025-11-19 | `SLST` | Solstice Advanced Materials | Remapped | → `SOLS` | yes |
| 2025-10-08 | `K` | Kellanova | Held (no match) | held `????` | yes |
| 2025-10-08 | `OLO` | Olo Inc | Held (no match) | held `????` | yes |
| 2025-10-06 | `COMM` | CommScope | Held (no match) | held `????` | yes |
