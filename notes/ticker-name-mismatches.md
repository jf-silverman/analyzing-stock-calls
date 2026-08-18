# Ticker / Company Name Mismatches — Review Queue

**66 ticker(s)** hold a company name that Yahoo Finance says belongs to
a different company. These are not all the same problem — some are wrong data,
most are a name we wrote informally — so they are split by **what can actually be
proved**, not by what they look like.

The test is the question in reverse: ignoring the symbol we filed it under, what
symbol does Yahoo return for the company name we stored? A different symbol back
means the call is sitting on the wrong company; the same symbol means our name is
merely informal.

That settles **10 of 66**. It cannot settle the other
**56**, because Yahoo's search only matches *current legal* names — it
returns nothing for "Snapchat", "Burlington Coat Factory" or "D-Wave Systems"
exactly as it returns nothing for a caption garble. Those need the transcript.

> **Generated file — do not edit by hand.**
>
> ```bash
> python3 code/pipeline.py --check-ticker-names
> ```
>
> **Manual — the nightly pipeline does not run this.** The nightly run checks only
> the episode it just analyzed and prints any flags in its output; this rebuilds the
> full picture across every ticker. Re-run it to pick up new episodes and to drop
> rows you have resolved.

Confirm against the transcript first, then retarget the mention (or delete it if it
duplicates a correct row):

```bash
sqlite3 data/mad_money.db \
  "UPDATE mentions SET ticker='CORRECT', closing_price=NULL WHERE ticker='WRONG';"
python3 code/pipeline.py --rebuild-shards
python3 code/pipeline.py --backfill-prices --tickers CORRECT
```

The **Where** column links each mention to its spot in the episode (`date · segment · timestamp`) so you can confirm the call by ear. A timestamp that resolves to the episode start means that section has no timing on disk yet — the same fallback the unknown-ticker queue uses.

**Said? sym/name** is two hints: did the **symbol** and did the **company name** each appear in the episode(s) it was mentioned in — `yes` / `✗` / `—` (no transcript). Read together they suggest where a bad row came from: `✗ / yes` = symbol inferred from a spoken name; `yes / ✗` = symbol was said but the name looks invented; `✗ / ✗` = neither was said (a caption garble, like Sterling's "STRL" heard as "SPRL"). **Weak hints, not verdicts.** They misfire in both directions: a correct pick spoken only as its ticker shows `yes / ✗` (the caller said "GDRX", never "GoodRx"), and a garble that became the stored name shows a false `yes` ("AEVEX" mis-heard as "Aeva"). A short symbol can also match a common word. And a handful of early-2026 episodes have a stale transcript in the DB, so their answer is unreliable. Worth a glance, nothing more.


## 1. Likely mis-tickers — 5 ticker(s), the data is wrong

**This is the section that matters.** Yahoo maps our stored company name to a
*different* symbol than the one we filed the call under, so the call is most
likely attached to an unrelated company and has inherited its price history.
Every return, chart and backtest for both tickers is affected.

The suggested symbol is advisory — Yahoo's search picks the first US listing and
can be wrong, and the *company* half of the pair may be the mistaken one.
Confirm against the transcript before changing anything.

| Ticker | We stored it as | That name is probably | But this symbol is | Said? sym/name | Where — date · segment · time | Transcript context |
|--------|-----------------|----------------------|--------------------|----------------|-------------------------------|--------------------|
| `LUMN` | **Lumentum** | `LITE` | Lumen Technologies, Inc. | **✗** / **✗** | 2025-12-16 · lightning_round · [35:02](https://youtu.be/E3aoDRmugAk?t=2102)<br>2026-08-12 · opening_commentary · [0:01](https://youtu.be/ieKCMqjaF-U?t=1) | …It is time. It's time for the lightning round. First of historical only seen the… |
| `AXIM` | **Voyager Technologies** | `VOYG` | AXIM Biotechnologies, Inc. | **✗** / yes | 2026-08-04 · interview · [12:00](https://youtu.be/M1Hwvz1qklQ?t=720) / [20:01](https://youtu.be/M1Hwvz1qklQ?t=1201) | …levitation on May money tonight. Voyager Technology is shooting for the moon to… |
| `FREL` | **Federal Realty Investment Trust** | `FRT` | Fidelity MSCI Real Estate Index ETF | **✗** / yes | 2025-12-19 · caller_qa · [0:03](https://youtu.be/1rEdy1i5kZk?t=3) | …travel trust, do you know that this is one of the best… |
| `THRM` | **Therav Bio** | `TBPH` | Gentherm Incorporated | **✗** / yes | 2025-12-17 · lightning_round · [34:40](https://youtu.be/-5TXDFR_roU?t=2080) | …So, is my stock the bio a buy sell or hold?… |
| `WF` | **Wells Fargo** | `WFC` | Woori Financial Group Inc. | **✗** / yes | 2025-12-19 · opening_commentary · [0:03](https://youtu.be/1rEdy1i5kZk?t=3) | …article about Wells Fargo in the journal. Goldman Sachs up 56% for the… |


## 2. Undecidable without the transcript — 56 ticker(s)

Yahoo's search recognises neither name, so there is no evidence either way. This
bucket genuinely mixes both problems: harmless old names ("Burlington Coat
Factory", "Snapchat") sit next to real mis-tickers ("Kagra" filed on Kinross
Gold, "Verdiv" on a Vanguard ETF). Read the transcript.

**`Similar?` is a weak triage hint, not a verdict.** `no` means the two names
share nothing and is worth looking at first; `~` means they resemble each other
and is worth looking at last. No string rule does better than this — "Inspira
Technologies" vs "Inspire Medical" are different companies but score like
"Snapchat" vs "Snap Inc", and "Eagle Gold" vs "Eagle Cement" share a word
exactly the way "D-Wave Systems" vs "D-Wave Quantum" do. Sorted hint-first.

| Ticker | We stored it as | Yahoo's name | Similar? | Said? sym/name | Where — date · segment · time | Transcript context |
|--------|-----------------|--------------|----------|----------------|-------------------------------|--------------------|
| `RH` | **Restoration Hardware** | RH | **no** | yes / yes | 2025-12-12 · in_depth_analysis · [20:35](https://youtu.be/jER6ZOPH_tA?t=1235)<br>2026-03-23 · closing_commentary · [40:18](https://youtu.be/tbKfYpp8OIg?t=2418)<br>2026-04-01 · in_depth_analysis · [9:40](https://youtu.be/6dJehtsavKU?t=580)<br>2026-04-08 · opening_commentary · [0:17](https://youtu.be/wYidr1VpMYI?t=17) | …stand? Uh, stocks like RH after the Fed's latest commentary. I'm digging… |
| `BP` | **British Petroleum** | BP p.l.c. | **no** | yes / **✗** | 2025-12-15 · lightning_round · [37:04](https://youtu.be/98hVLcFQtqM?t=2224)<br>2026-04-06 · opening_commentary · [0:17](https://youtu.be/fM_8kRzsk60?t=17) | …It's BP stock. You should sell it before you hang up… |
| `ABX` | **Barrick Gold** | Abacus Global Management, Inc. | **no** | **✗** / yes | 2026-01-28 · opening_commentary · [episode](https://youtu.be/UxsosXoIT9E) | …listen to me first about gold. It was a huge huge win again today. We just don't… |
| `ACOM` | **Acorn Realty Trust** | Harbor Active Commodity ETF | **no** | yes / yes | 2026-03-06 · lightning_round · [35:07](https://youtu.be/OLHPn2XxtzQ?t=2107) | …ACOM. Yeah, they they didn't have a good… |
| `AHCO` | **Acuity Electronics** | AdaptHealth Corp. | **no** | **✗** / yes | 2026-04-27 · in_depth_analysis · [11:11](https://youtu.be/HhPaoUmAoJA?t=671) / [23:57](https://youtu.be/HhPaoUmAoJA?t=1437) / [26:04](https://youtu.be/HhPaoUmAoJA?t=1564) | …position in Acuity Electronics. That is a DuPont spin-off that makes specialized… |
| `ATEN` | **Aten International** | A10 Networks, Inc. | **no** | **✗** / **✗** | 2026-05-05 · opening_commentary · [0:18](https://youtu.be/stBiW-NPi9E?t=18) | …Hey, I'm Kramer. Welcome to Mad Money. Welcome to Craig Friends. I'm just… |
| `BCTX` | **Billion to One** | BriaCell Therapeutics Corp. | **no** | **✗** / yes | 2026-04-27 · lightning_round · [33:04](https://youtu.be/HhPaoUmAoJA?t=1984) | …thoughts on the stock? Billion to one. We look into Billion to one. We think… |
| `BITO` | **Billion to One** | ProShares Bitcoin ETF | **no** | **✗** / yes | 2026-05-14 · lightning_round · [36:07](https://youtu.be/WubZMiGRH-I?t=2167) | …different. It's Billion to One. We like Billion to One. We looked at We… |
| `EQST` | **Equipment Share** | Energy Quest, Inc | **no** | **✗** / yes | 2026-01-26 · in_depth_analysis · [episode](https://youtu.be/NX6I2jJccas) | …Equipment Share, will be one of them. It's pretty compelling story. This is a… |
| `FG` | **Figure Technologies** | F&G Annuities & Life, Inc. | **no** | **✗** / yes | 2026-01-20 · in_depth_analysis · [episode](https://youtu.be/tKgYSl5KSq0) | …today, Galaxy Digital and Figure Technology, they're 44% and 76%… |
| `GWH` | **Global Wafers** | ESS Tech, Inc. | **no** | **✗** / yes | 2026-08-14 · interview · [0:39](https://youtu.be/ch0JZQ7PGh0?t=39) | …and then it you know it goes to global wafers a company that's in Texas and… |
| `IMRX` | **Immunity Bio** | Immuneering Corporation | **no** | **✗** / yes | 2026-06-16 · lightning_round · [36:38](https://youtu.be/2XCGYERvEzg?t=2198) | …Hey, so I've been following immunity bio for a while now and it seems to me… |
| `IMTX` | **Immunity Bio** | Immatics N.V. | **no** | **✗** / yes | 2026-04-17 · lightning_round · [36:27](https://youtu.be/HYRppgkEDXc?t=2187) | …My question is about Immunity Bio. I know it it really is.… |
| `INSP` | **Inspira Technologies** | Inspire Medical Systems, Inc. | **no** | yes / **✗** | 2026-04-20 · lightning_round · [35:06](https://youtu.be/-kPm8LikEBI?t=2106) | …stock is INSP in faction. I know. I know. Look, it's a power block… |
| `KD` | **Kendrell** | Kyndryl Holdings, Inc. | **no** | yes / **✗** | 2026-02-24 · lightning_round · [36:00](https://youtu.be/g09PyNhWRws?t=2160) | …trap. What do you think of ticker KD Kindrell?… |
| `KGC` | **Kagra** | Kinross Gold Corporation | **no** | **✗** / yes | 2026-03-27 · opening_commentary · [0:17](https://youtu.be/aK5g9aWVbWU?t=17) | …and that is Kagra. Now here's a stock that typifies what's been happening to… |
| `KMG` | **Kimcor** | KMG Chemicals, Inc. | **no** | **✗** / **✗** | 2026-03-26 · opening_commentary · [0:18](https://youtu.be/LPnGGW9Dm48?t=18) | …Hey, I'm Kramer. Welcome to Mad Money. Welcome to Cray America. I'll be with my… |
| `LAR` | **Lenar** | Lithium Argentina AG | **no** | yes / yes | 2025-12-18 · in_depth_analysis · [9:06](https://youtu.be/jvCnOf1NQTU?t=546) / [39:14](https://youtu.be/jvCnOf1NQTU?t=2354) | …that about from when we talk about LAR, but suffice it to say that the average… |
| `MDLM` | **Medline Industries** | MEDLEY MANAGEMENT INC | **no** | **✗** / yes | 2026-01-16 · lightning_round · [episode](https://youtu.be/3qf_h8DXLyY) | …My question is about Medline. I got in at $39.… |
| `NSP` | **NewScale Power** | Insperity, Inc. | **no** | **✗** / yes | 2025-12-15 · closing_commentary · [38:51](https://youtu.be/98hVLcFQtqM?t=2331) | …Meta is getting power plants and data centers, both of which could be frankly… |
| `NU` | **Nubank** | Nu Holdings Ltd. | **no** | **✗** / **✗** | 2026-01-23 · lightning_round · [episode](https://youtu.be/rzJfXrAjODY) | — |
| `OOK` | **One Oak** | OOK ETF | **no** | **✗** / yes | 2025-12-19 · lightning_round · [37:48](https://youtu.be/1rEdy1i5kZk?t=2268) | …your thoughts are on One Oak. Ticker symbol O O K.… |
| `PBR` | **Petrobras** | Petroleo Brasileiro S.A. Petrob | **no** | yes / **✗** | 2026-04-02 · lightning_round · [36:20](https://youtu.be/3nt_bL2oclU?t=2180) | …calling you about PBR. Um this stock has had quite a run. It's… |
| `PCG` | **PG&E Corporation** | Pacific Gas & Electric Co. | **no** | yes / yes | 2026-04-23 · interview · [10:51](https://youtu.be/4AOW-E3MQLY?t=651) / [21:05](https://youtu.be/4AOW-E3MQLY?t=1265) / [28:54](https://youtu.be/4AOW-E3MQLY?t=1734) | …PCG. Be patient, people. Thank you, Patti.… |
| `PDYN` | **Paladin** | Palladyne AI Corp. | **no** | yes / yes | 2026-03-24 · lightning_round · [36:15](https://youtu.be/WIdKqDtRhRg?t=2175) | …PDYN, Paladin. I bought it at about $9 a share and it's down to about six and a… |
| `PERM` | **Perpetua Resource Corporation** | Global X Permanent ETF | **no** | **✗** / yes | 2025-12-15 · lightning_round · [37:04](https://youtu.be/98hVLcFQtqM?t=2224) | …perpetua uh resource corporation. It's a hot stock. But let's go with… |
| `PLC` | **Power & Light Company** | Principal U.S. Large-Cap Multi-Factor ETF | **no** | yes / yes | 2026-07-06 · lightning_round · [36:30](https://youtu.be/UjZ1MYcw2OA?t=2190) | …So, uh looking at PLC, this is a $ 1.8 billion stock. Sales is nearly 700… |
| `RAN` | **Ramco Resources** | RanMarine Technology B.V. | **no** | yes / yes | 2026-01-27 · lightning_round · [episode](https://youtu.be/VqOMrFKSevs) | …for the time being. We ran out of time in the interview before I could ask him… |
| `SENT` | **Sentinel One** | AdvisorShares Alpha DNA Equity Sentiment ETF | **no** | **✗** / yes | 2025-12-18 · lightning_round · [35:01](https://youtu.be/jvCnOf1NQTU?t=2101) | …His own one-man war against inflation. a war I think he may have won before he's… |
| `SLS` | **Selecta Biosciences** | SELLAS Life Sciences Group, Inc | **no** | yes / **✗** | 2026-05-28 · lightning_round · [35:08](https://youtu.be/G_nPvcsM8LA?t=2108) | …All right. SLS. That's SLS.… |
| `THO` | **Tenneco (Thomas Oil)** | THOR Industries, Inc. | **no** | **✗** / yes | 2026-06-23 · opening_commentary · [0:17](https://youtu.be/TW-oqAAfZDY?t=17) | …the Thomas, an oil company with a real gusher in Indonesia. Oh, I didn't have… |
| `TMPO` | **Tempest AI** | Tempo Automation Holdings, Inc. | **no** | **✗** / yes | 2026-03-27 · lightning_round · [35:00](https://youtu.be/aK5g9aWVbWU?t=2100) | …see if it has any future Tempest AI. I like this stock, but the problem is… |
| `VDV` | **Verdiv** | Vanguard Developed Markets ex-U | **no** | **✗** / **✗** | 2026-03-27 · lightning_round · [35:00](https://youtu.be/aK5g9aWVbWU?t=2100) | …the sky's the limit. It's a fast fire lightning round next.… |
| `SNAP` | **Snapchat** | Snap Inc. | ~ | yes / yes | 2026-04-01 · opening_commentary · [0:17](https://youtu.be/6dJehtsavKU?t=17)<br>2026-04-28 · lightning_round · [35:35](https://youtu.be/Lgj13qHO9bE?t=2135)<br>2026-06-16 · opening_commentary · [0:17](https://youtu.be/2XCGYERvEzg?t=17) | …question about Snapchat Inc. And yeah, just how you feel overall about the… |
| `AMC` | **AMC Theaters** | AMC Entertainment Holdings, Inc. | ~ | yes / yes | 2026-07-10 · closing_commentary · [45:00](https://youtu.be/vM-QdKTx9n8?t=2700)<br>2026-07-30 · lightning_round · [36:40](https://youtu.be/Z6eL97pcmJ0?t=2200) | …meme stock guys pushed AMC, the movie theater chain, as a turnaround play, I… |
| `BK` | **BNY Mellon** | Bank of New York Mellon Corp | ~ | **✗** / **✗** | 2025-12-17 · opening_commentary · [0:05](https://youtu.be/-5TXDFR_roU?t=5)<br>2026-06-23 · opening_commentary · [0:17](https://youtu.be/TW-oqAAfZDY?t=17) | …[music] Hey, I'm Kramer. Welcome to Mad Money.… |
| `BTC` | **Bitcoin** | Grayscale Bitcoin Mini Trust (B | ~ | **✗** / yes | 2026-03-02 · closing_commentary · [40:25](https://youtu.be/FiJ8qLa09no?t=2425)<br>2026-04-24 · lightning_round · [37:08](https://youtu.be/i3xD9jEIuDg?t=2228) | …investors turn to gold and Bitcoin. So, what's in the cards for the latter? We… |
| `BURL` | **Burlington Coat Factory** | Burlington Stores, Inc. | ~ | **✗** / yes | 2026-04-07 · opening_commentary · [0:01](https://youtu.be/8at2Eyt89RQ?t=1)<br>2026-08-05 · closing_commentary · [40:11](https://youtu.be/aiwQyVmb800?t=2411) | …Ross and Burlington. These have been phenomenal performers leaving their… |
| `HBAN` | **Huntington Bancorp** | Huntington Bancshares Incorpora | ~ | **✗** / yes | 2026-01-16 · lightning_round · [episode](https://youtu.be/3qf_h8DXLyY)<br>2026-04-10 · am_i_diversified · [episode](https://youtu.be/6tmhL98Xa1g) | …Huntington Bank, Nvidia, and Walmart. Jim, am I… |
| `UAMY` | **U.S. Antimony Corporation** | United States Antimony Corporation | ~ | yes / **✗** | 2026-01-20 · in_depth_analysis · [episode](https://youtu.be/tKgYSl5KSq0)<br>2026-03-12 · lightning_round · [35:12](https://youtu.be/MsK1NxlzwvY?t=2112) | …years. U.S. Anemone, UAMY. [music]… |
| `VCX` | **Fundrise Innovation Fund** | Fundrise Growth Tech Fund, LLC | ~ | yes / yes | 2026-03-24 · in_depth_analysis · [31:10](https://youtu.be/WIdKqDtRhRg?t=1870)<br>2026-07-08 · in_depth_analysis · [11:21](https://youtu.be/k1DEekxlGG4?t=681) / [19:20](https://youtu.be/k1DEekxlGG4?t=1160) | …came public with a bang. After VCX uh uh shares opened for trading at $31.25 last… |
| `AEIS` | **Array Electronic Industries (Ametek/AEI Systems)** | Advanced Energy Industries, Inc | ~ | yes / yes | 2026-03-05 · lightning_round · [29:00](https://youtu.be/5dAxKTZ3sIA?t=1740) | …on AEIS for a while now. The stock seems to be trading at a premium. I'm… |
| `ATAI` | **ATAI Life Sciences** | AtaiBeckley Inc. | ~ | **✗** / **✗** | 2026-08-06 · lightning_round · [36:44](https://youtu.be/GZGcGzB0WRI?t=2204) | …It is time. [music]… |
| `AUTR` | **Auterion** | Autris | ~ | **✗** / **✗** | 2026-03-27 · investing_club_meeting · [episode](https://youtu.be/aK5g9aWVbWU) | — |
| `DECK` | **Deckers Brands** | Deckers Outdoor Corporation | ~ | **✗** / yes | 2026-01-05 · in_depth_analysis · [29:31](https://youtu.be/k4bEI8CxAgQ?t=1771) | …hot shoe it fell 49% in 2025. Deckers got paxed earlier the year on tariff… |
| `EAGLE` | **Eagle Gold Mining** | Eagle Cement Corp | ~ | yes / yes | 2026-01-28 · opening_commentary · [episode](https://youtu.be/UxsosXoIT9E) | …like Eagle. They were on the other night, second largest gold miner because… |
| `EQPT` | **Equipment Shares** | EquipmentShare.com Inc | ~ | **✗** / yes | 2026-03-24 · lightning_round · [36:15](https://youtu.be/WIdKqDtRhRg?t=2175) | …I'm calling about equipment shares, E Q P T.… |
| `FWONK` | **Liberty Media Formula One** | Liberty Media Corporation - Ser | ~ | yes / yes | 2026-03-06 · in_depth_analysis · [17:27](https://youtu.be/OLHPn2XxtzQ?t=1047) / [28:14](https://youtu.be/OLHPn2XxtzQ?t=1694) | …trade under this ticker FWONK. I call it FWonk because they have the… |
| `KRMN` | **Karman Space & Defense** | Karman Holdings Inc. | ~ | **✗** / yes | 2026-01-05 · lightning_round · [36:40](https://youtu.be/k4bEI8CxAgQ?t=2200) | …card space. I think this bank, which trades at just 12 times earnings, is the… |
| `NRGV` | **Energy Storage Company** | Energy Vault Holdings, Inc. | ~ | yes / yes | 2026-05-15 · lightning_round · [36:39](https://youtu.be/JIQheuNzaAI?t=2199) | …NRGV. Well, that's a pure spec that's losing a… |
| `QBTS` | **D-Wave Systems** | D-Wave Quantum Inc. | ~ | **✗** / yes | 2026-06-01 · lightning_round · [36:06](https://youtu.be/oHrBBaAh4Jc?t=2166) | …Nibbius, and Core Wave getting a nice shout out and then users which will be… |
| `RAL` | **Reliant** | Ralliant Corporation | ~ | yes / yes | 2026-06-05 · lightning_round · [36:00](https://youtu.be/LDtdnZddg-k?t=2160) | …equipment space. I'm calling about RAL Reliant.… |
| `SATL` | **Satalogic Inc.** | Satellogic Inc. | ~ | yes / yes | 2026-01-28 · lightning_round · [episode](https://youtu.be/UxsosXoIT9E) | …ticker symbol SATL. Yeah, this thing is a parabolic move. When I see a parabolic… |
| `SGHC` | **Supergroup** | Super Group (SGHC) Limited | ~ | yes / yes | 2026-04-13 · interview · [29:50](https://youtu.be/ZJNO3IbjSDg?t=1790) | …SGHC, what can I say? Mad Money's back after the break.… |
| `TEM` | **Tempest AI** | Tempus AI, Inc. | ~ | yes / yes | 2026-02-06 · opening_commentary · [0:03](https://youtu.be/xpEBYrOyHPE?t=3) | …ticker TEM. Well, you know, we're not recommending… |
| `TKR` | **Timkin** | Timken Company (The) | ~ | yes / yes | 2026-06-04 · interview · [10:28](https://youtu.be/2KJ4PtpX3Wk?t=628) / [29:43](https://youtu.be/2KJ4PtpX3Wk?t=1783) | …Tim Cubby, TKR. Wish I hadn't lost track of [music] you guys because what a great… |


## 3. Name variants — 5 ticker(s), cosmetic only

Yahoo maps our stored name back to the *same* symbol, so the ticker is correct
and no price history is affected. Our name is just informal ("Snapchat"),
shortened ("Petco"), dated ("Burlington Coat Factory"), or a caption
misspelling. Safe to leave alone; fix only if the wording bothers you on the
site. For a genuine rename, prefer `New Name (formerly Old Name)` — see the
renamed-companies note in CLAUDE.md.

| Ticker | We stored it as | Yahoo's name | Said? sym/name | Where — date · segment · time | Transcript context |
|--------|-----------------|--------------|----------------|-------------------------------|--------------------|
| `CMG` | **Chipotle** | Chipotle Mexican Grill, Inc. | yes / yes | 2025-12-12 · closing_commentary · [39:47](https://youtu.be/jER6ZOPH_tA?t=2387)<br>2025-12-31 · in_depth_analysis · [10:28](https://youtu.be/xuWprgXUOlM?t=628)<br>2026-01-27 · opening_commentary · [episode](https://youtu.be/VqOMrFKSevs)<br>2026-01-30 · opening_commentary · [episode](https://youtu.be/LmFPOWVLnwo)<br>2026-02-03 · interview · [9:57](https://youtu.be/ZVTZc0j4hDY?t=597) / [23:38](https://youtu.be/ZVTZc0j4hDY?t=1418)<br>2026-03-06 · opening_commentary · [0:22](https://youtu.be/OLHPn2XxtzQ?t=22) / [10:00](https://youtu.be/OLHPn2XxtzQ?t=600)<br>2026-03-17 · opening_commentary · [0:17](https://youtu.be/5l7fzwpaiiQ?t=17)<br>2026-03-27 · opening_commentary · [0:17](https://youtu.be/aK5g9aWVbWU?t=17)<br>2026-04-17 · interview · [13:03](https://youtu.be/HYRppgkEDXc?t=783)<br>2026-04-24 · opening_commentary · [0:17](https://youtu.be/i3xD9jEIuDg?t=17)<br>2026-06-29 · opening_commentary · [0:17](https://youtu.be/oYqPlJRTd9o?t=17) / [10:46](https://youtu.be/oYqPlJRTd9o?t=646) / [27:06](https://youtu.be/oYqPlJRTd9o?t=1626) / [33:06](https://youtu.be/oYqPlJRTd9o?t=1986)<br>2026-07-29 · interview · [10:27](https://youtu.be/dV7Dw4bIv7M?t=627) / [28:17](https://youtu.be/dV7Dw4bIv7M?t=1697) | …Chipotle, but not so terrifying to the data center plays with the super… |
| `DD` | **DuPont** | DuPont de Nemours, Inc. | **✗** / yes | 2025-12-12 · lightning_round · [35:00](https://youtu.be/jER6ZOPH_tA?t=2100)<br>2026-02-06 · opening_commentary · [0:03](https://youtu.be/xpEBYrOyHPE?t=3) | …Jim, I've owned DuPont for 20 years. Uh after the recent spin-off of Quinity and… |
| `FDS` | **FactSet** | FactSet Research Systems Inc. | **✗** / yes | 2026-02-23 · in_depth_analysis · [episode](https://youtu.be/Ij0nyL7Z2vc) | …financial data like FactSet or Fair Isaac, that's the keeper of FICO, uh or… |
| `PHM` | **Pulte Homes** | PulteGroup, Inc. | **✗** / yes | 2026-05-26 · lightning_round · [35:46](https://youtu.be/UBZvilR6Zuo?t=2146) | …homes, upscale condos, and active adult communities. I still own it. I think… |
| `WOOF` | **Petco** | Petco Health and Wellness Compa | yes / yes | 2026-06-05 · opening_commentary · [0:17](https://youtu.be/LDtdnZddg-k?t=17) | …woof, which might also I mean, I got noises for every single one of these… |


_Checked 886 tickers with a stored company name. Tickers Yahoo does not
recognise at all (hallucinated, private, OTC) are not listed here — see the
'Hallucinated tickers' note in CLAUDE.md._

_This list intentionally over-flags. A shared single word is not treated as a
match, so `Chipotle` vs `Chipotle Mexican Grill` appears even though it is fine —
the same rule is what keeps `Marriott Vacations Worldwide` from matching
`Marriott International`. Missing a real mis-ticker costs a corrupted price
history; a false positive costs one glance._
