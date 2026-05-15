# ForgeYields Risk Methodology — v3.10 Amendment

**Date:** 2026-05-08
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** Layer 2 §A4 Collateral Backing — RWA Backing Class Taxonomy
**Backwards compatibility:** v3.9 §A1-A3, A5 unchanged; only §A4 RWA rubric receives a class-specific overlay
**Trigger:** sUSDat/srUSDat backing rescore (Scenario C, 2026-05-08) revealed that the pre-v3.10 unified RWA rubric under-priced corporate-preferred-equity and reinsurance-tranche backing patterns vs. T-Bill backing.

---

## 1. Overview

The pre-v3.10 §A4 RWA rubric treats all "Real-World Asset backing" in a single unified rubric (bands 1–10 based on enforceability, liquidation timeline, mark-to-market, custody). In practice, three structurally distinct backing classes exist with materially different risk profiles:

- **Class T — T-Bill / Sovereign Debt**: USDC, USDG, $M (M^0), USDat, RLUSD, sUSDS
- **Class C — Corporate Credit / Preferred Equity**: STRC (sUSDat backing), corporate bonds, structured corporate credit
- **Class I — Insurance / Reinsurance Tranche**: reUSDe (junior reinsurance), CAT bonds, Nayms-tranche tokens

v3.10 introduces a **class-specific overlay** to the existing A4 rubric that:
1. Establishes class-specific score floors
2. Establishes class-specific cap floors via Rule 3 interaction
3. Mandates cross-dependency flags for Class C/I per §3.0
4. Provides class-specific worked examples for calibration

---

## 2. Motivation — what we encountered in production

### 2.1 Scenario C rescore (2026-05-08)

The sUSDat L2 backing rescore exposed a structural ambiguity in the unified RWA rubric:

| Reading | A4 score | sUSDat ARS |
|---------|----------|------------|
| Original (unified RWA rubric, "partially opaque OR < 20% illiquid" interpretation) | 5 | 5.20 (APPROVED 30%) |
| Strict reading per liquidation/MtM/custody dimensions | 7 | 6.10 (WATCHLIST 15%) |

**0.90 ARS shift on rescore** — substantive enough to warrant explicit rubric guidance. The unified rubric's "5–6: partially opaque OR < 20% illiquid" band absorbed STRC's structurally riskier pattern (perpetual no-maturity + self-reported NAV + offshore custody + corporate junior cap stack) without flagging it as Class C.

### 2.2 Pre-v3.10 ambiguity

The v3.9 RWA rubric reads:

> 5–6: Reserves ≥ 100% but semi-annual attestation OR partially opaque OR < 20% illiquid
> 7–8: Under-collateralized OR attestation > 6 mo old OR 20–50% illiquid reserves

This is **dimension-specific** (attestation cadence, opacity, illiquidity %) but **class-agnostic**. STRC backing matches the dimension-specific 7–8 band on liquidation timeline + mark-to-market dimensions but could be argued at 5–6 on "partially opaque" reading. The lack of class-specific guidance creates assessor judgment slack that is inappropriate for a hard cap-bearing methodology.

### 2.3 Risk asymmetry

Class T (T-Bill) and Class C (Corporate Pref Eq) have fundamentally different stress recovery profiles:

| Class | Bankruptcy waterfall | Liquidation timeline | Mark-to-market | Custody concentration |
|-------|----------------------|----------------------|----------------|------------------------|
| T (T-Bill) | Sovereign debt = paid first | T+0 to T+1 | Daily Bloomberg/FRED | GSIB tier |
| C (Corp Pref Eq) | Senior to common, junior to all debt | 30-90+ days, perpetual no-maturity in worst case | Monthly issuer NAV | Broker-dealer + SPV |
| I (Reins/CAT) | Catastrophe-event-linked | 90+ days, years for tail events | Actuarial models | Insurance regulator + offshore SPV |

A unified rubric that treats these as comparable would systematically under-price Class C and Class I exposures. v3.10 corrects this.

---

## 3. The RWA Backing Class Taxonomy

### 3.1 Class T — T-Bill / Sovereign Debt

**Definition:** Backing consists of short-duration sovereign debt (US Treasuries, OECD sovereign equivalents) held in regulated custody, with daily mark-to-market against deep liquid secondary markets.

**Structural characteristics:**
- Cap stack: sovereign debt holders paid first (top of any waterfall)
- Liquidation timeline: T+0 to T+1 institutional rails
- Mark-to-market: daily Bloomberg / Reuters / FRED prices; market depth $10T+
- Custodian: GSIB tier (BNY Mellon, State Street, Fireblocks integrated, JPM)
- Stress recovery: sovereign CDS markets liquid; Treasury secondary depth >$1T daily volume

**Examples in registry:**
- USDC (Circle, ARS 2.50, A4=2)
- USDG (Paxos, ARS 3.30, A4=3)
- $M (M^0 Foundation, A4=3 implicit)
- USDat (Saturn → $M, ARS 4.40, A4=3.5)
- RLUSD (Ripple SCTC, ARS 3.15, A4=3)
- sUSDS (Sky Protocol, ARS 3.25, A4 = 3 implicit; multi-collateral CDP heritage but sovereign-backed)

### 3.2 Class C — Corporate Credit / Preferred Equity

**Definition:** Backing consists of corporate credit instruments (preferred equity, corporate bonds, structured corporate notes) where the issuer's solvency determines recovery.

**Structural characteristics:**
- Cap stack: senior to common equity, junior to all corporate debt (incl. convertibles, term loans, secured debt)
- Liquidation timeline: 30-90+ days secondary market sale; perpetual instruments have NO maturity (forever subject to issuer call); bankruptcy waterfall = months/years
- Mark-to-market: monthly to quarterly issuer NAV reports; thin secondary market for most instruments
- Custodian: broker-dealers (Clear Street, Pershing) + SPV / fund vehicles (BVI, Cayman, Delaware Series LLC)
- Stress recovery: depends on issuer solvency; correlated with macro / sector / commodity factors

**Examples in registry:**
- STRC backing of sUSDat (Strategy Inc. perpetual preferred equity, A4=7 post-Scenario-C rescore)
- syrupUSDC backing (Maple corporate borrowers — multiple institutional, classifiable as Class C diversified)
- syrupUSDT backing (similar Class C diversified)
- Lombard structured credit (sub-classified Class C)

### 3.3 Class I — Insurance / Reinsurance Tranche

**Definition:** Backing consists of insurance-treaty premium, CAT (catastrophe) bonds, or reinsurance-tranche tokens where catastrophe events or industry-wide losses can wipe out senior or junior holders.

**Structural characteristics:**
- Cap stack: junior tranches absorb first-loss; senior tranches buffered up to attachment point; CAT bonds may be wiped on triggering event
- Liquidation timeline: 90+ days for normal redemption; years for catastrophe-event-linked tail exposure
- Mark-to-market: actuarial models, infrequent (quarterly to annual)
- Custodian: insurance regulator (Lloyd's of London, Bermuda Monetary Authority) + offshore SPV
- Stress recovery: catastrophe event or industry default may wipe out tranches; recovery dependent on insurance industry solvency

**Examples in registry:**
- reUSDe (Re Protocol, junior reinsurance tranche, ARS 7.55 EXCLUDED)
- jrUSDe (Strata Junior USDe, junior tranche structural EXCLUDED, ARS 6.75)

---

## 4. A4 — Class-Specific Rubrics

### 4.1 Class T standard rubric (UNCHANGED from v3.9)

| Score | Class T criteria |
|-------|------------------|
| 1–2 | 100%+ liquid audited reserves, real-time PoR, monthly Big-4 attestation, GSIB regulated custodian |
| 3–4 | 100%+ reserves, quarterly attestation by reputable firm, disclosed composition, regulated custodian |
| 5–6 | Reserves ≥ 100% but semi-annual attestation OR partially opaque OR < 20% illiquid |
| 7–8 | Under-collateralized OR attestation > 6 mo old OR 20–50% illiquid reserves |
| 9–10 | No attestation OR significantly under-collateralized OR > 50% illiquid/opaque OR no independent verification |

### 4.2 Class C overlay (NEW v3.10)

**Score floor: 5** for any Class C exposure (cannot score below 5 regardless of issuer credit quality).

| Score | Class C criteria |
|-------|------------------|
| 5–6 | Investment-grade rated issuer (S&P/Moody's BBB+ or higher), monthly issuer NAV reports, GSIB-tier custody, liquidation 30-60 days, deep secondary market, Big-4 monthly attestation |
| **7–8** | **Default Class C reading**: untested or non-IG issuer credit, monthly or less frequent NAV, broker-dealer custody (not GSIB), liquidation > 60 days, thin secondary market, no Big-4 monthly |
| 9–10 | Distressed corporate credit (CCC or below), infrequent NAV, offshore unregulated SPV, illiquid secondary, post-bankruptcy waterfall, undisclosed exposure |

**Class C cross-dependency mandatory** per §3.0 active-operations rule: Class C backing IS the issuer's active operation (corporate credit cycle management). Flag cross-dep with L1 C5/C4; apply more conservative; do not double-penalize.

### 4.3 Class I overlay (NEW v3.10)

**Score floor: 7** for any senior Class I exposure; **floor: 9** for junior / first-loss Class I.

| Score | Class I criteria |
|-------|------------------|
| **7–8** | Senior reinsurance tranche, established reinsurer (Lloyd's syndicate, Bermuda Re-Re), regulated jurisdiction, actuarial models published, attachment point > 50% |
| **9–10** | **Default junior Class I reading**: Junior tranche / first-loss / catastrophe-event-linked, opaque actuarial models, offshore SPV, attachment point < 50% OR no documented attachment |

**Class I cross-dependency mandatory** per §3.0; same logic as Class C.

---

## 5. Cap floors via Rule 3 interaction

When ARS computed with Class C / I rubric falls in APPROVED or WATCHLIST bands:

| Class | Band | Cap |
|-------|------|-----|
| T | per Rule 3 standard | per Rule 3 (1.0–3.0 unlimited; 3.1–5.0 60%; 5.1–6.0 30%; 6.0–6.5 WATCHLIST 15%) |
| **C — APPROVED (ARS ≤ 6.0)** | per Rule 3 standard | per Rule 3 standard |
| **C — WATCHLIST band (6.0–6.5)** | WATCHLIST | **15% per vault** (no relaxation) |
| **I — senior (ARS ≤ 6.5)** | per Rule 3 with Class I floor | **15% per vault floor** regardless of Rule 3 band |
| **I — junior (default)** | EXCLUDED structurally | not eligible |

**v3.10 clarification on monitoring cadence:**
- Class T APPROVED: quarterly L2 re-score
- Class C APPROVED: **monthly L2 re-score required** (vs quarterly for Class T)
- Class C WATCHLIST: **monthly L2 re-score** + documented monitoring triggers
- Class I senior: **monthly L2 re-score** + actuarial event monitoring

---

## 6. Class-specific monitoring triggers

### 6.1 Class C mandatory monitoring (v3.10)

Per L2 assessment file, must document specific triggers for each Class C asset:
1. Issuer credit rating changes (S&P/Moody's/Fitch)
2. Issuer dividend / coupon miss event
3. Issuer financial covenant breach
4. Mark-to-market oracle change (e.g., transition to/from Chainlink)
5. Custodian counterparty risk events (broker-dealer SIPC events, SPV legal challenges)
6. Macro / sector stress (e.g., for STRC: BTC -40%+ stress test on Strategy Inc.)

### 6.2 Class I mandatory monitoring (v3.10)

1. Catastrophe event triggers (named storm landfall, earthquake, insurance industry losses > attachment point)
2. Reinsurer credit events
3. Actuarial model updates
4. Tranche attachment point migration

---

## 7. Worked examples

### 7.1 USDat (Class T) — calibration anchor

```
Backing chain: USDat → $M → US Treasuries (M^0 permissioned Minters at Fireblocks GSIB-integrated custody)
Class: T (sovereign debt)
A4 standard rubric: band 3-4 ("Enforceable claim in one jurisdiction, weekly independent valuation, regulated custodian")
A4 = 3.5 (no Big-4 monthly specifically on USDat holdings; M^0 18mo zero-incident track record)
Class T cap: per Rule 3 standard
ARS = 4.40 → 30% per vault (band 3.1-5.0)
```

### 7.2 sUSDat (Class C, STRC backing) — post-Scenario-C rescore

```
Backing chain: sUSDat → STRC (Strategy Inc. perpetual preferred equity) → Clear Street custody → Securitize transfer agent → BVI fund
Class: C (corporate preferred equity)
Issuer credit: Strategy Inc. NOT investment-grade (BTC-leveraged)
NAV oracle: Saturn-operated (self-reported); Chainlink "future"
Custody: Clear Street (broker-dealer, not GSIB) + BVI fund (offshore)
Liquidation: STRC perpetual no-maturity → > 90 days strict reading
A4 Class C overlay: band 7-8 ("Default Class C reading: untested issuer credit, monthly NAV, broker-dealer custody, > 60 days, thin secondary, no Big-4 monthly")
A4 = 7
Class C WATCHLIST cap floor: 15% per vault (binding once ARS in WATCHLIST band)
ARS = 6.10 → WATCHLIST 15% per vault
Monthly L2 re-score required.
```

**Re-score upward conditions** (per Class C monitoring):
- Strategy Inc. achieves investment-grade rating (S&P BBB+) → A4 6
- Big-4 monthly attestation specifically on sUSDat holdings → A4 6
- Chainlink STRC oracle live → A4 6
- 12-month STRC dividend cycle without miss + above conditions → A4 5

### 7.3 reUSDe (Class I junior) — existing exclusion confirmed

```
Backing chain: reUSDe → Re Protocol junior reinsurance tranche → BVI SPV
Class: I (insurance/reinsurance, junior first-loss)
Tranche: junior (first-loss, catastrophe-event exposure)
A4 Class I junior overlay: band 9-10 ("Default junior Class I reading")
A4 = 8 (band 7-8 to 9-10 boundary, conservative)
Existing assessment ARS 7.55 → EXCLUDED (consistent with Class I junior structural exclusion)
v3.10 ratifies existing exclusion with explicit class-specific rationale.
```

### 7.4 syrupUSDC (Class C diversified) — existing assessment review

```
Backing chain: syrupUSDC → Maple → 28+ institutional borrowers (corporate credit diversified)
Class: C (corporate credit diversified)
Mitigations vs sUSDat single-issuer: 28+ borrower diversification + Maple's 2yr+ track record + active loan book monitoring
A4 Class C with diversification credit: band 5-6 (mitigations push toward IG-equivalent)
Existing A4 = 4 → may warrant +1 to 5 under v3.10 strict reading
ARS would shift 4.20 → 4.40 (still APPROVED 50% via L1 cascade Maple PRS 3.65)
**v3.10 forward-looking — do not retroactively rescore syrupUSDC unless triggered by Maple-level event.**
```

---

## 8. Cross-dependency rule (§3.0 v3.10 update)

**Pre-v3.10 §3.0 active-operations rule:**
> "If backing IS the issuer's active operation (e.g., delta-neutral hedging, RWA invoice management) rather than static reserves (cash, T-bills, ETH in a contract), score A4 normally **and** flag a cross-dependency with L1 C5/C4. Apply the more conservative of the two — do not double-penalize."

**v3.10 clarification:**
- For **Class T** assets, the active-operations rule is OPTIONAL (T-Bill custody is structural, not "operations")
- For **Class C** assets, the active-operations rule is **MANDATORY** (corporate credit IS the issuer's business operation; cross-dep flag required)
- For **Class I** assets, the active-operations rule is **MANDATORY** (insurance underwriting IS the issuer's business operation)

Cross-dep flag format: "L1 [issuer] C5/C4 cross-dep flagged via Class C/I active-operations rule per v3.10 §8."

---

## 9. Audit pass on existing registry (2026-05-08)

### 9.1 Class T assets — no rescore needed (v3.10 forward-looking)

| Asset | Current ARS | Current A4 | Class T applies | Verdict |
|-------|-------------|------------|------------------|---------|
| USDC | 2.50 | 2 | ✓ | unchanged |
| USDG | 3.30 | 3 | ✓ | unchanged |
| RLUSD | 3.15 | 3 | ✓ | unchanged |
| sUSDS | 3.25 | 3 | ✓ (multi-collateral CDP heritage; sovereign exposure dominant) | unchanged |
| BOLD | 4.05 | 4 | ✓ (asset-collateralized via ETH; not corporate credit) | unchanged |
| USDat | 4.40 | 3.5 | ✓ | unchanged |
| sUSDe | 4.25 | 5 | mixed (T-Bill backing + delta-neutral CEX hedging — Class T base + Class C operational layer) | flag for review |

### 9.2 Class C assets — Scenario C already applied

| Asset | Pre-v3.10 ARS | Post-v3.10 ARS | A4 | Verdict |
|-------|---------------|----------------|------|---------|
| sUSDat | 5.20 | **6.10** | 7 (Class C strict) | WATCHLIST 15% (rescored 2026-05-08) |
| srUSDat | 5.23 | **6.30** | 7 (Class C strict) | WATCHLIST 15% (rescored 2026-05-08) |
| syrupUSDC | 4.20 | review next quarterly | 4 → maybe 5 | unchanged pending review |
| syrupUSDT | 5.05 | review next quarterly | 6 | unchanged pending review |

### 9.3 Class I assets — existing exclusion ratified

| Asset | Tranche | Current ARS | Class I | Verdict |
|-------|---------|-------------|---------|---------|
| reUSDe | junior | 7.55 | ✓ junior | EXCLUDED (ratified) |
| jrUSDe | junior | 6.75 | ✓ junior | EXCLUDED (ratified) |
| jrUSDat | junior | structural EXCLUDED | ✓ junior (cascade) | EXCLUDED (ratified) |

### 9.4 No verdict changes from v3.10 beyond Scenario C

v3.10 primarily formalizes the Scenario C rescore framework + provides forward-looking guidance for future Saturn-pattern, structured-credit, and reinsurance-tranche assessments.

---

## 10. Conservative-by-design rationale

The v3.10 amendment maintains the methodology's "fail conservatively" principle:

1. **Class C floor at band 5** prevents understating corporate-credit risk even with investment-grade issuers
2. **Class C default at band 7-8** captures uninvestigated/non-IG corporate credit accurately
3. **Class I junior floor at band 9-10** confirms structural exclusion of first-loss reinsurance exposure
4. **Mandatory cross-dep flag** for Class C / I prevents missing the L1 cycle-correlation risk
5. **Monthly re-score cadence** for Class C / I (vs quarterly Class T) reflects the higher cycle-volatility of these classes
6. **No relaxation** of existing v3.7/v3.8/v3.9 hard caps — v3.10 only tightens

The framework is **forward-looking** for future RWA assessments and **operationally aligned** with how risk frameworks in the broader industry (LlamaRisk, Gauntlet, Steakhouse) treat structured credit and reinsurance exposure.

---

## 11. Implementation checklist

- [x] `methodology/v310_amendment.md` — this document
- [ ] `methodology/risk_methodology.md` §A4 — add Class T/C/I taxonomy reference
- [ ] `methodology/risk_methodology.md` §3.0 — update active-operations rule with Class-specific MANDATORY flag
- [ ] `methodology/risk_methodology.md` §A4.4.4 / §A4.4.5 — add Class C and Class I overlay subsections
- [ ] `methodology/risk_methodology.md` §7 Changelog — add v3.10 entry
- [ ] `methodology/risk_methodology.md` Rule 3 cap table — add Class C WATCHLIST + Class I floors
- [ ] Update version headers v3.9 → v3.10
- [ ] Sync v3.10 amendment to Notion methodology parent
- [ ] Update sUSDat / srUSDat L2 assessments to reference v3.10 Class C explicitly
- [ ] Update PT-srUSDat L3 assessment to reference v3.10 Class C cascade
- [ ] Update reUSDe / jrUSDe / jrUSDat assessments to reference v3.10 Class I (ratification)

---

## 12. Approval

Authored by Risk Analyst Agent under CEO direction, 2026-05-08. Triggered by empirical observation during the Scenario C sUSDat/srUSDat backing rescore (2026-05-08): the methodology lacked an explicit RWA backing class taxonomy that distinguishes T-Bill-backed assets (USDat) from corporate-preferred-equity-backed assets (sUSDat / STRC) from reinsurance-tranche-backed assets (reUSDe / jrUSDat). The unified rubric created assessor judgment slack inappropriate for hard-cap-bearing assessments.

**Conservative-by-design properties preserved:**
- Tail-risk pricing strengthened (Class C floor 5; default 7-8; Class I junior floor 9-10)
- No relaxation of existing hard caps (ARS > 6.5 EXCLUDED; A2 ≥ 9 EXCLUDED)
- Cross-dependency flags strengthened (mandatory for Class C / I)
- Monitoring cadence tightened (monthly for Class C / I vs quarterly for Class T)
- Cap floors strengthened (Class C WATCHLIST 15%; Class I senior 15%; Class I junior structurally EXCLUDED)
- Forward-looking — does not retroactively re-score existing assessments beyond Scenario C (already applied)

Approved for v3.10 release: 2026-05-08.

---

## 13. Forward-looking research questions

The v3.10 taxonomy raises open questions for future amendments:

1. **Class C diversification credit** — should multi-issuer Class C (e.g., syrupUSDC's 28+ borrowers) receive an explicit credit vs single-issuer Class C (sUSDat's STRC)? Quantify in v3.11+.
2. **Class T with active-operations overlay** — sUSDe (T-Bill base + delta-neutral CEX hedging) is mixed-class. Should v3.11 introduce "Class TM" (T-Bill modified by active operations) explicitly?
3. **Class C investment-grade transition triggers** — should an issuer's IG rating downgrade trigger automatic L2 re-score? Specify mechanical triggers in v3.11.
4. **Class I CAT bond senior** — currently no senior Class I exposure in registry; specify worked examples for future onboarding.
5. **Cross-class composition** — multi-asset strategies that combine Class T + Class C exposure (e.g., Curve LP USDC/sUSDat) — does paired-asset weighting need a class-specific adjustment?

These are research questions — v3.10 establishes the taxonomy framework; v3.11+ may refine.
