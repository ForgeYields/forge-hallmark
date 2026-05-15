# ForgeYields Risk Methodology — v3.7 Amendment

**Date:** 2026-05-06
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** Layer 2 asset scoring (A3 rubric + hard caps + concentration cap structure)
**Backwards compatibility:** v3.6 L1 rules unchanged; L2 rubrics A1, A2, A4, A5 unchanged

---

## 1. Overview

v3.7 introduces two coupled changes to the L2 (asset) scoring framework:

1. **A3 Async Exit Credit** — formalize credit for documented withdrawal mechanisms (queue, cooldown, atomic redeem) in the A3 (Liquidity Depth) rubric.
2. **WATCHLIST band 6.0–6.5** — raise the ARS hard cap from > 6.0 to > 6.5, introducing a graduated WATCHLIST tier with 15% per-vault concentration cap.

Both changes apply uniformly to all L2 assets and are documented in `risk_methodology.md` §3.3 (A3) and §4.2.1 / §5 Rule 3 respectively.

---

## 2. Motivation — the asymmetric application problem

### 2.1 Pre-v3.7 implicit treatment of redemption paths

Pre-v3.7, A3 was strictly DEX/CEX market microstructure. Empirically this produced asymmetric outcomes:

| Asset | DEX deep? | Async withdrawal? | A3 score (pre-v3.7) | Notes |
|-------|-----------|-------------------|---------------------|-------|
| sUSDS (Sky) | yes (atomic PSM redeem) | yes (atomic) | 2 | Atomic redeem implicitly credited via DEX-deep aggregator pricing |
| sUSDe (Ethena) | yes ($4.75B+ ecosystem) | yes (7-day cooldown) | 3 | DEX alone scores at floor; cooldown invisible |
| weETH (ether.fi) | yes ($158M 24h volume) | yes (NFT ~2 days) | 3 | Same pattern |
| syrupUSDC (Maple) | moderate ($842K + Instant Liquidity DEX path $15M) | yes (queue typical <2d, max 30d) | 5 | Queue partially credited via combined slippage |
| ynETHx (YieldNest) | thin ($1M Curve pool, $5.8K 24h volume) | yes (FIFO 1-10 days) | 9 | Queue NOT credited; A3 catastrophic |

**Asymmetric outcome:** assets with deep DEX got implicit queue credit (queue is redundant when DEX works), while DEX-thin assets with strong queues were over-penalized despite functional exit capability for patient holders.

### 2.2 Combinatorial clustering at ARS 6.0–6.5

Empirically, multiple L2 assets with mid-band penalties (A3=8/9, A4=6/7, A5=7/8) clustered just above the 6.0 hard cap without any single criterion hitting band 9+. The 6.0 binary threshold treated assets like ynETHx (ARS 6.60, structurally fragile but functional) the same as assets like USD0++ (ARS 7.40, mechanism-broken) — coarsening the signal in a way that obscured actionable risk grades.

L1 already addresses analogous combinatorial clustering with watchlist treatment for PRS 5.6–7.0; L3 with GRS 7.5 cap. L2 had no graduated band, forcing a binary EXCLUDED/APPROVED decision at 6.0.

---

## 3. Amendment 1 — A3 Async Exit Credit

### 3.1 Rubric modifier

For assets with documented async withdrawal mechanisms, apply the following A3 modifier:

| Mechanism class | Disclosed SLA | A3 credit |
|-----------------|---------------|-----------|
| Atomic redeem (no wait) | 0 | −2 bands |
| Queue / cooldown / NFT | ≤ 7 days | −1 band |
| Queue / cooldown / NFT | ≤ 14 days | −1 band conditional on no pause history |
| Queue / cooldown / NFT | > 14 days OR pause history within 12 months | no credit |

### 3.2 Eligibility requirements

To receive A3 credit, the mechanism must satisfy ALL of:

- Documented in protocol docs with disclosed SLA
- On-chain verifiable contract (queue contract address, redemption function selector traceable)
- No pause exceeding the stated SLA in the past 12 months
- Asset issuer L1 not in EXCLUDED status (cascade rule)

### 3.3 Caps on credit

- A3 cannot be reduced below band 3 by async credit alone (preserves DEX/CEX failure-mode signal)
- Maximum total credit: −2 bands
- Credit nullified retroactively if a pause event occurs (asset re-scored on next monthly cadence)

### 3.4 Why these numbers

- **Atomic redeem (sUSDS) at −2 bands** formalizes the credit already implicit in current scoring (sUSDS A3=2 reflects atomic PSM redeem, not DEX depth alone)
- **7-day SLA at −1 band** matches the standard institutional unwind cycle (monthly rebalance, weekly liquidity checks)
- **14-day SLA at −1 band conditional** reflects that beyond 7 days, the queue's protective value degrades; conditional on no pause history because pauses indicate the queue itself is fragile
- **Floor at band 3** preserves the "DEX/CEX failure mode" signal — even with strong queue, if both queue AND DEX fail simultaneously (peer LRT depeg precedent: ezETH April 2024), A3 should still reflect non-trivial liquidity risk

### 3.5 Tail-risk preservation

The credit does NOT eliminate tail-risk pricing because:
- Both queue and DEX can fail simultaneously in stress (well-documented for LRT category)
- The floor-band-3 cap ensures even fully-credited assets retain liquidity-risk signal
- Pause history nullification ensures track-record-broken assets lose credit
- A1 still independently scores the redemption mechanism (no double credit)

---

## 4. Amendment 2 — WATCHLIST band 6.0–6.5

### 4.1 Hard cap revision

Pre-v3.7: `ARS > 6.0 → EXCLUDED`
v3.7:     `ARS > 6.5 → EXCLUDED`; `ARS 6.0–6.5 → WATCHLIST`

### 4.2 Concentration cap structure (Rule 3)

| ARS | Cap | Status |
|-----|-----|--------|
| 1.0–3.0 | No limit | APPROVED |
| 3.1–5.0 | 60% | APPROVED |
| 5.1–6.0 | 30% | APPROVED |
| **6.0–6.5** | **15%** | **WATCHLIST (v3.7)** |
| > 6.5 | Not eligible | EXCLUDED |

### 4.3 WATCHLIST operational requirements

Assets in the 6.0–6.5 band require:

1. **Monthly re-scoring cadence** (vs. quarterly for standard APPROVED)
2. **Documented monitoring trigger list** in the L2 assessment file:
   - Specific NAV deviation thresholds (e.g., >2% sustained >4h)
   - Mechanism pause events (queue, redemption, oracle staleness)
   - Mcap drop thresholds (e.g., −20% additional)
   - Pool/liquidity drop thresholds (e.g., DEX TVL −30%)
   - L1 issuer downgrade events
3. **Quarterly CEO review** with documented hold/exit decision recorded in registry
4. **Position sizing within Type-specific Rule 4 caps** still applies and may bind tighter than the 15% Rule 3 cap

### 4.4 WATCHLIST cascade behavior

- L3 strategies using a WATCHLIST asset are NOT auto-excluded; they apply the 15% cap normally
- If a WATCHLIST asset cascades to EXCLUDED via L1 (issuer downgrade) or A2 (depeg event), strategies cascade to EXCLUDED
- WATCHLIST classification does NOT trigger §4.2.1 forced unwind; it triggers cap-compliance check (any over-cap excess must be unwound within 30 days of the rescore)

### 4.5 Why a watchlist band, not a cap shift

A pure cap shift (ARS > 6.0 → ARS > 6.5) would treat assets at 6.4 the same as assets at 5.5. This loses the combinatorial-fragility signal. The watchlist band preserves the signal via the tighter 15% cap + monitoring overhead, while allowing tactical deployment in cases where structural concerns are real but not catastrophic.

---

## 5. Audit Pass on Existing Registry (2026-05-06)

### 5.1 Methodology

For each L2 asset with ARS in the 4.5–7.0 range, applied:
1. A3 async exit credit per v3.7 §3.1
2. Re-cascade ARS calculation
3. New verdict per v3.7 §4.1 (cap structure)

### 5.2 Affected assets (changes only)

| Asset | Pre-v3.7 ARS | A3 (old) | A3 (v3.7) | Post-v3.7 ARS | Pre-v3.7 verdict | v3.7 verdict | Cap |
|-------|--------------|----------|-----------|----------------|------------------|--------------|-----|
| **ynETHx** | 6.60 | 9 | **8** (−1, FIFO 1-10d) | **6.35** | EXCLUDED | **WATCHLIST** | 15% |
| **ynRWAx** | 6.20 | (n/a) | (n/a, no async path) | 6.20 | EXCLUDED | **WATCHLIST** subject to RWA legal-enforceability override | 15% (or override-EXCLUDED) |
| **syrupUSDC** | 4.45 | 5 | **4** (−1, queue <2d) | 4.20 | APPROVED | APPROVED | 50% (unchanged, L1 cascade binding) |

### 5.3 Confirmed unchanged (no async credit applies; over 6.5 cap)

| Asset | ARS | Verdict | Reason |
|-------|-----|---------|--------|
| apyUSD | 6.60 | EXCLUDED | L1 APYX cascade-excluded (C3=9), independent of L2 |
| triBTC | 6.70 | EXCLUDED | LP unwind friction + small-cap concentration; >6.5 |
| jrUSDe | 6.75 | EXCLUDED | Junior tranche structural; >6.5 |
| xsBTC | 6.95 | EXCLUDED | Bridge + custody concerns; >6.5 |
| alETH | 7.20 | EXCLUDED | Sustained peg discount + L1 cascade |
| USD0++ | 7.40 | EXCLUDED | Bond-like with redemption gating + L1 cascade |
| reUSDe | 7.55 | EXCLUDED | Junior reinsurance tranche |
| dgnETH | 7.85 | EXCLUDED | Higher-complexity LRT structural |
| pmUSD | 8.30 | EXCLUDED | RAAC mechanism failure |
| rsETH | 9.20 | EXCLUDED | Multiple structural issues |

### 5.4 Net registry impact

- **+2 assets to WATCHLIST** (ynETHx, ynRWAx)
- **0 assets EXCLUDED → APPROVED** (no false-positive flips)
- **0 assets APPROVED → EXCLUDED** (no false-negative regressions)
- **1 asset cap unchanged** (syrupUSDC ARS 4.45 → 4.20, both fall in 3.1–5.0 band → 60% Rule 3 cap; binding L1 cap 50% unchanged)

### 5.5 ynRWAx note (RWA override)

ynRWAx has structural concerns beyond the A3/ARS scoring (RWA legal enforceability, daily valuation cadence, jurisdictional exposure) that the v3.7 amendment does not address. The watchlist auto-classification under v3.7 §4 should be reviewed against the RWA-specific A4 override rubric before confirming the WATCHLIST status. If A4 RWA criteria flag `7-8` or worse, an issuer-level override should hold ynRWAx EXCLUDED despite the v3.7 watchlist eligibility.

---

## 6. Backtest — does v3.7 catch what v3.6 missed?

### 6.1 Validation cases

**ynETHx (target case for v3.7):**
- Pre-v3.7: ARS 6.60 → EXCLUDED (no path)
- v3.7: ARS 6.35 → WATCHLIST 15% with documented monitoring triggers
- Outcome: matches reality — asset has functional FIFO queue exit (1-10 days NAV par) but real structural concerns (compound LRT, mcap decline -78%, AI-rebal opaque)

**syrupUSDC (sanity check):**
- Pre-v3.7: ARS 4.45 → APPROVED, cap 50% via L1 Maple cascade
- v3.7: ARS 4.20 → APPROVED unchanged
- Outcome: queue credit recognized but does not flip verdict (asset was already APPROVED); calibration tighter

**sUSDS, sUSDe, weETH (no-change cases):**
- All already at A3 floor (band 3 or lower)
- v3.7 atomic-redeem credit (−2 bands for sUSDS) does not lower below floor → A3 unchanged
- ARS unchanged; verdict unchanged
- Outcome: amendment is non-disruptive for established APPROVED assets

### 6.2 Counter-cases (intentional non-flips)

**xsBTC, triBTC, jrUSDe (ARS 6.7–6.95):**
- All EXCLUDED by structural issues that A3 credit does not address (bridge custody, LP unwind friction, junior tranche structure)
- ARS post-v3.7 still > 6.5 → EXCLUDED preserved
- Outcome: amendment correctly does NOT rescue these assets

**rsETH, pmUSD, USD0++ (ARS 7.4+):**
- Far above WATCHLIST band; no eligible A3 credit could move them under 6.5
- EXCLUDED preserved
- Outcome: amendment correctly does NOT rescue these assets

### 6.3 No false-positive flips

Audit pass confirmed zero assets flipped APPROVED → EXCLUDED or EXCLUDED → APPROVED inappropriately. The only verdict changes are within the 6.0–6.5 band where the WATCHLIST tier was specifically designed to apply.

---

## 7. Operational impact

### 7.1 fyETH ynETHx position

Pre-v3.7 status: §4.2.1 mandate to exit within 7 days.
v3.7 status: WATCHLIST 15% cap.

**Effective treatment of fyETH's ~63 ETH ynETHx position:**
- Position size relative to fyETH TVL: must verify current % vs new 15% cap
- If position > 15% of fyETH: partial unwind via FIFO queue (NAV par, 1-10 days) to compliance
- If position ≤ 15% of fyETH: hold subject to monthly re-scoring + monitoring triggers
- Replaces the §4.2.1 forced-exit deadline with a position-sizing compliance check

### 7.2 Future strategies on WATCHLIST assets

L3 strategies using a WATCHLIST asset:
- May be APPROVED if GRS ≤ 7.5 standard threshold met
- Must respect the 15% L2 cap on the asset (often the binding cap)
- Receive WATCHLIST monitoring inheritance — strategy reviews coordinate with L2 monthly rescore cadence

### 7.3 BD/LP communication

WATCHLIST assets in the registry should be presented as:
- "Asset shows structural fragilities at the asset level (specific drivers documented in L2 assessment)"
- "Eligible for tactical deployment under tightened 15% concentration cap"
- "Subject to monthly re-scoring with pre-defined monitoring triggers"
- "Distinguishes graduated risk from binary APPROVED/EXCLUDED"

This framing strengthens the rigor narrative (vs. coarsening to binary APPROVED/EXCLUDED) and provides a richer set of risk grades for institutional LP discussions.

---

## 8. Implementation checklist

- [x] §3.3 A3 rubric: Async Exit Credit modifier added with eligibility, credit table, caps
- [x] §4.2.1: ARS hard cap raised from > 6.0 to > 6.5; WATCHLIST band 6.0–6.5 referenced
- [x] §5 Rule 3: Asset concentration cap table updated with WATCHLIST band 6.0–6.5 / 15% cap
- [x] §7 Changelog: v3.7 entry added
- [x] `layer2_asset_assessment_methodology.md` §8: hard cap section updated with WATCHLIST treatment
- [x] `methodology/v37_amendment.md` (this file) — comprehensive amendment documentation
- [ ] Phase 2: Audit pass on registry — re-score affected assets (ynETHx, ynRWAx, syrupUSDC) and update `asset_registry.json`
- [ ] Phase 3: Notion sync — update L2 index with WATCHLIST status, create v3.7 changelog page

---

## 9. Approval

This amendment was authored by the Risk Analyst Agent under CEO direction (forge.fi.contact@gmail.com), date 2026-05-06. The amendment was driven by an empirical observation during fyETH ynETHx position management: the strict pre-v3.7 hard cap forced an exit decision on an asset with functional exit paths and only mid-tier structural concerns, while assets with deep DEX implicitly received queue credit. This mismatch was resolved by formalizing the credit and introducing graduated exposure for borderline assets.

The amendment is conservative-by-design:
- Tail-risk pricing is preserved via floor-band-3 cap on A3 credit
- WATCHLIST is operationally heavier than APPROVED (monthly rescore + triggers + CEO review)
- 15% cap is materially tighter than the standard 30% for ARS 5.1-6.0
- No assets escape EXCLUDED status if their drivers are not amenable to A3 credit

Approved for v3.7 release: 2026-05-06.
