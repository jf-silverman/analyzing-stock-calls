# Ticker / Company Identification — Error Taxonomy

Every Mad Money mention has to resolve a spoken reference into a **(ticker, company,
as-of date)** triple. This file enumerates every way that resolution can go wrong,
because the failure modes are not interchangeable: some corrupt price history silently,
some are cosmetic, and some are not errors at all but *real-world changes* that our data
model has to represent over time.

It is the companion to three other files:

- **`ticker-name-mismatches.md`** — the generated queue of *currently* suspect rows.
- **`unknown-tickers.md`** — the generated queue of `????` placeholders.
- **`ticker-review-log.md`** — the hand-written record of specific decisions already made.

This file is the *why* behind those: the classes of error, how each is detected, which
direction its hints point, and — for the corporate-action classes — the change-tracking
mechanism proposed at the bottom.

A recurring theme: **a mention is a pair, and either half can be the wrong one.** The
stored *ticker* can be wrong while the *name* is right (`WF` for Wells Fargo), the *name*
can be wrong while the ticker is right (`PBR` stored as "Polarcoin"), or — the dangerous
case — *both* can be wrong (`AVX` stored as "Aeva Technologies", truly AEVEX/`AVEX`).

---

## A. Transcript / caption failures
*The reference is already corrupted before the model ever sees it. YouTube auto-captions
are the source.*

**A1. Company name garbled to a homophone.**
The caption mis-transcribes a spoken company name. The model then does its best with a
wrong string.
- Examples: "Newor" → Nucor, "Sanders" → SanDisk, "Wirehouser" → Weyerhaeuser,
  "Anamoney" → Antimony, "Lumenum" → Lumentum, "Kagra" → (a gold miner), "Therav" →
  Theravance, "travel trust" → *Charitable Trust* (not a company at all).
- Detection: name won't resolve on Yahoo, or resolves to an unrelated symbol.

**A2. Spoken ticker letters mis-transcribed.**
The caller/host spells a ticker and the caption gets a letter wrong.
- Examples: "STRL" → "SPRL" (Sterling Infrastructure), "AEVEX" heard and written "AX".
- Detection: the garbled symbol often doesn't exist, or is a tiny unrelated stock.

**A3. Garble collides with a *real, unrelated* ticker (the silent one).**
The mis-transcription happens to be a valid symbol for a different company, so nothing
looks wrong — the call inherits that company's entire price history, chart, and backtest.
- Examples: "NWR"/`NWR` instead of `NUE`; `SNPS` (Synopsys) instead of `SNDK`; `BWX`
  (a **bond ETF**) instead of `BWXT`/`BW`; `FREL` (a Fidelity **ETF**) instead of `FRT`.
- Detection: **this is the class the mismatch queue exists for** — Yahoo maps the stored
  *name* to a different symbol than the one we filed under.
- Cost: highest. Two tickers are corrupted (the wrong one gains a bogus row; the right
  one is missing a real one).

**A4. Segment/timing garble misattributes the call.**
Adjacent lightning-round callers blur together, so a call lands on the wrong caller's
stock, or two calls collapse into one.
- Examples: the GDRX-vs-SPRL back-to-back callers; "United States Antimony up 85% /
  Critical Metals 147%" read as one garbled mention (two separate sell calls).
- Detection: only by reading/listening to the surrounding transcript.

**A5. Wrong episode's transcript stored (infrastructure bug).**
Not a caption problem but a pipeline one: `episodes.transcript_text` held a *different*
episode's text (BUG-014, 3 episodes). Any provenance check ("was the symbol said?") is
then meaningless.
- Detection: the "Said? sym/name" provenance columns disagree with reality across a whole
  episode. Resolved by `--resync-transcripts`; re-run if a whole date looks off.

---

## B. Model (Haiku) inference failures
*The caption may be fine; the model picks the wrong symbol or invents a name.*

**B1. Plausible-but-wrong symbol from a correct name.**
The model hears the right company and guesses a symbol that belongs to someone else.
- Example: "Barrick Gold" → `ABX` (a stale/recycled symbol, now Abacus Global).
- Detection: mismatch queue; provenance reads `✗ symbol / yes name`.

**B2. Invented name to fit a garbled symbol (both halves wrong).**
The caption garbles the symbol, and the model *hallucinates a company name* that matches
the garble — so neither half is trustworthy and the suggestion engine points confidently
at the wrong company.
- Example: `AVX` → stored name "Aeva Technologies" (a real, *held* stock) when the truth
  was AEVEX/`AVEX`. Applying the "obvious" fix would have corrupted a holding.
- Detection: hardest. The name resolves cleanly on Yahoo — to the wrong company. Only a
  human who knows the actual company catches it.

**B3. Same-theme ETF symbol instead of the company's own.**
The model reaches for a thematic ETF ticker that shares the company's sector/buzzword.
- Examples: Quantinuum → `QTUM` (Defiance Quantum ETF); Qnity → `QTEC` (First Trust Tech
  ETF); "Bitcoin" → `BITO`/`BTC`; a rare-earth name → a commodity ETF.
- Detection: Yahoo says the symbol is a fund, not a company; price scale looks like an ETF.

**B4. Fully hallucinated symbol (no listing anywhere).**
~368 of the DB's tickers have no Yahoo data at all — invented for small/obscure/private
names. Distinct from placeholders: here the model *asserts* a wrong symbol.
- Detection: Yahoo returns nothing; excluded from most analytics automatically.

**B5. Placeholder emitted (`????` / `???`).**
Not an error — the model correctly *declines* to guess. Needs a human to identify (or to
confirm it's a private company that will never have a symbol).
- Detection: `is_unknown_ticker()`; tracked in `unknown-tickers.md`.

**B6. Non-call promoted to a call.**
Passing market color ("a positive WSJ article about Wells Fargo"; "JP Morgan is my #2
bank after Wells Fargo") gets tagged with a sentiment as if it were a discrete pick.
- Detection: transcript shows no actual buy/sell recommendation; the reference is
  comparative or contextual. Judgment call — delete vs. keep as a soft rating.

**B7. One company, several distinct calls conflated (or vice versa).**
A ticker mentioned in two analysis passes, or under an empty-string segment, can leave
duplicate rows that `UNIQUE(episode_id, ticker, segment)` doesn't catch (empty segment is
a distinct value). Or one issuer's calls get split across two symbols.
- Example: the `BWX`/`BWXT`/`BW` episode — four rows, two companies, two passes.
- Detection: count rows per (episode, ticker); look for empty-segment twins.

---

## C. Corporate actions — time-dependent identity
*Not transcription or model errors. The company genuinely changed, so **(ticker, company)
is only correct as of a date.** These are the cases that need the change-tracking
mechanism in Section E, because the "right" answer depends on when the call aired.*

**C1. Rename, same ticker.**
Company changes its name but keeps the symbol.
- Examples: MicroStrategy → Strategy (`MSTR`); Facebook → Meta.
- Handling today: store `New Name (formerly Old Name)`; `names_agree()` is alias-aware so
  either name finds the ticker. See CLAUDE.md "Renamed companies".

**C2. Re-ticker, same company.**
Company keeps its name/business but changes its trading symbol.
- Example: Barrick Gold kept the name, moved `GOLD`/`ABX` → `B`.
- Handling: needs a dated alias — a call *before* the change should resolve the old
  symbol's price, a call *after* the new one.

**C3. Rename **and** re-ticker.**
Both change at once. Two aliases to track from one effective date.

**C4. Symbol recycled to a different company.**
A freed-up ticker is later assigned to an unrelated company, so the *same string* means
different companies in different years — and Yahoo's search returns the **current** owner.
- Examples: `ABX` (Barrick → Abacus Global Management); `GOLD`.
- Detection trap: this is why Yahoo's symbol lookup can "confirm" a wrong answer. Always
  check the name Yahoo returns *for that symbol*, not just the symbol.

**C5. Split into multiple similar-named entities.**
A parent spins off / splits into several companies whose names share a root, so a call
about one can be filed on another.
- Example: DuPont → DuPont de Nemours + Qnity Electronics (+ historical Chemours,
  Corteva). "DD" vs "Qnity/`Q`" is easy to cross.
- Detection: multiple live tickers share a name stem; match the *business described*, not
  the stem.

**C6. Merger / acquisition.**
Two symbols collapse into one; the acquired ticker is delisted as of the close date. A
call before the deal is about a company that no longer trades after it.

**C7. IPO / private→public transition.**
No price exists before the IPO date; the first print is often an IPO-day pop.
- Examples: SpaceX/`SPCX` (IPO 2026-06-12), AEVEX/`AVEX` (IPO-day $26.93), SK Hynix ADR
  `SKHY` (US line only from 2026-07-10), Anthropic/`ANTH` & OpenAI/`OPAI` (still private).
- Handling: `_is_private(ticker, date)` skips price fetch pre-IPO; site shows "(private)".

**C8. Delisting / bankruptcy.**
Ticker stops trading; late-date price fetches return "possibly delisted, no data" (the
routine Yahoo misses seen every run for `ALTS`, `CTRA`, etc.).

**C9. Stock split / reverse split.**
Not an identity change but a **price-series discontinuity**. A naive stitch of pre- and
post-split closes produces a phantom cliff.
- Example: CRWD 4:1 (BUG-015). Handled by fetching a single split-adjusted range and the
  14-day `--reconcile-prices` pass.

**C10. Dual listing / ADR vs. local line; share classes.**
One company, several symbols: an ADR vs. the home-market line, or multiple share classes.
- Examples: SK Hynix (Korea line vs. `SKHY` ADR); `GOOG`/`GOOGL`; Liberty tracking stocks
  (`FWONK` vs. siblings); `BRK.A`/`BRK.B`.
- Detection: pick the class/line Cramer actually referenced; prices differ between them.

---

## D. Speaker errors (Cramer or a guest)
*The audio itself is wrong, so no transcript or lookup fix helps — only knowledge of the
company.*

**D1. Wrong ticker, right name (or vice versa).** A caller or host misstates the symbol
but names the company correctly, or names the wrong company but gives the right symbol.
As you noted, it's unlikely both are wrong — usually one is right and can anchor the fix,
or one is simply **omitted** (only the name is spoken, or only the ticker).

**D2. Old/pre-change symbol spoken.** A caller uses a symbol that was correct years ago
(overlaps with C2/C4). The as-of date decides which is meant.

---

## E. Reference / tooling failures
*Our own resolution machinery introduces the error.*

**E1. Yahoo returns a stale/historical symbol.** (See C4.) Yahoo search matches historical
ticker associations; `suggest_ticker_detail()` guards this by also checking the returned
*name*.

**E2. Yahoo returns nothing for an informal/old/foreign name.** Produces false
"undecidable" rows — "Snapchat", "Burlington Coat Factory", "D-Wave Systems" look exactly
like a caption garble to the search. This is why Section 2 of the mismatch queue can't be
auto-resolved.

**E3. Cross-store reconciliation overwrites a correct value.** Trusting `stock_sentiments.json`
over the DB (or vice versa) has corrupted names before (`ACM`, `BLK`, `CDNS`, `LRCX`).
Rule: **Yahoo arbitrates; neither store owns.** See CLAUDE.md "Company names".

**E4. Provenance hints misread.** The "Said? sym/name" columns cry wolf in both directions
(a pick spoken only as its ticker reads name-absent; a garble that became the stored name
reads name-present). Advisory only, never automatic.

---

## F. Proposed: a dated ticker/company history

The corporate-action classes (**C1–C4, C6, C10**) all share one need: **the correct
(ticker, company) for a mention depends on the date it aired.** Today we encode only the
crudest version — `New Name (formerly Old Name)` in the stored name string — which handles
C1 (rename, same ticker) but nothing date-sensitive.

Proposal: a small, hand-maintained `data/ticker_history.json` (main-owned, like the other
generated/curated data files), keyed by a **stable entity id**, listing dated segments:

```json
{
  "barrick": {
    "canonical_name": "Barrick Mining",
    "segments": [
      { "from": null,         "to": "2019-01-01", "ticker": "ABX",  "name": "Barrick Gold" },
      { "from": "2019-01-02", "to": "2025-xx-xx", "ticker": "GOLD", "name": "Barrick Gold" },
      { "from": "2025-xx-xx", "to": null,         "ticker": "B",    "name": "Barrick Mining" }
    ],
    "actions": [
      { "date": "2025-xx-xx", "type": "rename+reticker", "note": "GOLD→B, Gold→Mining" }
    ]
  }
}
```

`type` ∈ `rename | reticker | rename+reticker | split | merge | ipo | delist | split_adjust
| adr_listing`. A resolver would, given a spoken name/symbol and an air date, return the
segment covering that date — so a pre-2019 "Barrick Gold" call resolves `ABX`'s prices and
a 2026 one resolves `B`'s, automatically.

**Scope note / recommendation:** most of our volume is a single ~7-month window
(Jan–Aug 2026), so few calls actually straddle a corporate action *today*. Start small —
record the handful we've already hit (Barrick, MicroStrategy→Strategy, SK Hynix ADR,
DuPont/Qnity, the private→public IPOs) as documentation in this file, and only build the
JSON + resolver if the backfill toward 2024 makes straddling calls common. `PRIVATE_COMPANIES`
+ `_is_private(ticker, date)` is already a working prototype of exactly this date-aware
pattern for the IPO case (C7); the history table generalizes it.

---

## Quick reference — which file catches which class

| Class | Silent? | Caught by |
|-------|---------|-----------|
| A3 garble→real ticker | **yes** | mismatch queue §1 (name→different symbol) + ingest validator |
| A1/A2 garble→dead symbol | no | Yahoo returns nothing; validator flags |
| B1 wrong symbol, right name | **yes** | mismatch queue §1 |
| B2 both wrong | **yes, worst** | human knowledge only |
| B3 ETF-for-company | partly | Yahoo says "fund"; price scale |
| B4 hallucinated | no | no Yahoo data; excluded from analytics |
| B5 placeholder | no | `unknown-tickers.md` |
| B6 non-call | **yes** | transcript read; judgment |
| C1–C10 corporate actions | varies | Section F history (proposed) + `_is_private` |
| E1/E2 Yahoo quirks | — | name-check guard in `suggest_ticker_detail()` |
