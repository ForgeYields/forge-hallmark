# ForgeYields Risk Methodology — v3.8 Amendment

**Date:** 2026-05-07
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** Layer 3 strategy scoring (S2 Collateral Composition + Multi-Protocol Rule when collateral is itself a Type 1/2/3 strategy receipt token)
**Backwards compatibility:** v3.7 L1/L2 rules unchanged; only L3 §6.2 (Type 3 lending) gets a new "Recursive Strategy Collateral Rule"

---

## 1. Overview

v3.8 closes a gap identified during v3.7 audit of Type 3 lending strategies: the methodology pre-v3.8 does not explicitly handle the case where the collateral in a lending market is itself an L3 strategy receipt token (Pendle PT/YT/LP, AMM LP tokens, lending receipt tokens like aTokens / yvTokens / gteUSDC).

**Pre-v3.8 ambiguity:** the S2 mapping table requires "weighted-avg L2 ARS" but Pendle PTs and similar are explicitly NOT L2 (per L2 methodology §0.1, they are L3 strategies). This created an undefined behavior gap.

**v3.8 resolution:** introduce an explicit "Recursive Strategy Collateral Rule" that handles the case via:
1. Multi-Protocol N+1 inflation
2. S2 mapping based on the underlying fundamental L2 ARS
3. S2 floor of 4 (recognizes composition complexity)
4. S1 +1 band penalty for recursive composition risk
5. Cascade-EXCLUDE if the underlying L3 strategy is itself EXCLUDED

---

## 2. Motivation — what we encountered in production

During the May 2026 ForgeYields strategy review, several real-world cases surfaced where this gap matters:

### 2.1 Real cases observed
- Morpho Blue PT-sUSDe-XXAUG2026 / USDC market (Pendle PT collateral)
- Morpho Blue earnAUSD/USDC Monad market (yield-bearing wrapper, falls back to L2 since wrapper is L2-eligible)
- Hypothetical Morpho Blue yvUSDC / USDC market (Yearn V3 receipt collateral)

### 2.2 The classification boundary

**Yield-bearing stablecoin wrappers** (sUSDS, syrupUSDC, sUSDe, earnAUSD, pufETH, msY) are explicitly L2 per §0.1 — covered by existing rules. Their dual-assessment satisfies S2 mapping naturally.

**Strategy-receipt tokens** (Pendle PT/YT/LP, AMM LP tokens, lending receipts) are explicitly L3 per §0.1 — NOT covered by current S2 mapping which requires L2 ARS input.

### 2.3 The risk asymmetry

When a Type 3 lending market accepts a strategy-receipt token as collateral:
- The collateral has its own strategy mechanism risk (oracle, IL, mechanism failure, queue mechanics)
- The collateral's underlying asset has L2 risk
- The lending market itself has its own protocol layer

A naive scoring that only uses the underlying L2 ARS would understate the recursive composition risk. A naive scoring that excludes such markets entirely would over-restrict the strategy universe (since composition is widely used in production DeFi).

---

## 3. Amendment — Recursive Strategy Collateral Rule

### 3.1 Trigger conditions

The rule applies when ANY of the following holds for a Type 3 lending strategy:
- The collateral asset is a Pendle PT, YT, or LP token
- The collateral asset is an AMM LP token (Curve, Balancer, Uniswap V3/V4)
- The collateral asset is a lending receipt token (aToken, cToken, yvToken, MetaMorpho ERC-4626 receipt that is NOT classified L2 per §0.1)

### 3.2 Scoring adjustments

When trigger conditions are met:

#### 3.2.1 Multi-Protocol Rule N+1 inflation
The L3 strategy/protocol issuing the collateral receipt counts as an additional protocol in the N count.

```
Example: Morpho Blue PT-sUSDe / USDC market
  N=2 protocols counted:
    P1 = Morpho Blue (lending venue, PRS 1.87)
    P2 = Pendle (PT issuer, PRS 4.24)
  ProtocolRisk = max(1.87, 4.24) + 0.20 × mean(1.87, 4.24)
              = 4.24 + 0.20 × 3.06
              = 4.85
```

Note: Underlying L2 issuer (e.g., Ethena for sUSDe used in PT-sUSDe) is still excluded per §4.2.2 (captured via L2 dual-assessment).

#### 3.2.2 S2 Collateral Composition mapping
Use the underlying fundamental L2 ARS for S2 input mapping, NOT a derived synthetic from the L3 GRS.

```
Example: PT-sUSDe / USDC market
  Underlying fundamental L2 = sUSDe (ARS 4.25)
  S2 mapping(4.25) = band 3.0-4.5 → S2 = 4
```

For multi-asset L3 collateral (e.g., AMM LPs):
- Use the paired-asset rule: S2 = mapping(0.75 × max(Ai) + 0.25 × mean(Ai))
- Where Ai are the L2 ARS of each token in the LP

#### 3.2.3 S2 floor (composition complexity)
**S2 = max(computed S2, 4)** — a minimum floor of 4 applies to recognize that recursive composition introduces non-trivial complexity even when the underlying is high-quality (e.g., USDC).

```
Example: Hypothetical Morpho Blue yvUSDC / USDC market
  Underlying fundamental L2 = USDC (ARS 2.50) → mapping → S2 = 2
  S2 floor 4 applies → S2 = 4
```

#### 3.2.4 S1 +1 band penalty
The L3 strategy holding strategy-receipt collateral receives a +1 band penalty on S1 (Risk Management Quality) to capture the recursive oracle risk, mechanism risk, and audit-coverage gap.

```
Example: PT-sUSDe / USDC market
  S1 base = 4 (Morpho Blue oracle + LLTV + IRM standard)
  S1 +1 recursive penalty = 5
```

If S1 base would be 9 (already at penalty band), the +1 penalty caps at 9.

### 3.3 Cascade rules

1. If the **underlying L3 strategy is EXCLUDED**, the lending market is cascade-EXCLUDED via L2 hard cap (the strategy receipt token cannot be used).

2. If the **underlying L3 strategy is APPROVED-or-WATCHLIST**, the lending market is eligible for L3 scoring under v3.8 rules.

3. If the **L3 strategy issuer protocol is EXCLUDED at L1** (e.g., Pendle EXCLUDED hypothetically), the lending market is cascade-EXCLUDED.

### 3.4 Documentation requirements

For any L3 lending strategy invoking v3.8 §3.1 trigger:
- Document the full composition chain (lending venue → strategy receipt → underlying L2 → underlying issuer L1)
- Cite the multi-protocol N count explicitly
- Show the S2 floor application if it binds
- Show the S1 +1 penalty application
- Identify any cascade gates explicitly

---

## 4. Worked Example — Morpho Blue PT-sUSDe-13AUG2026 / USDC market

### 4.1 Composition chain
```
Morpho Blue (lending venue, PRS 1.87)
  ↓ market accepts:
PT-sUSDe-13AUG2026 (Pendle PT, Type 2C strategy receipt, NOT L2)
  ↓ underlying:
sUSDe (Ethena, L2 ARS 4.25)
  ↓ underlying:
USDe (L2 ARS 3.70)
  ↓ underlying L1:
Ethena (PRS 4.42)
```

### 4.2 Score computation (v3.8 rules)

**Multi-Protocol Rule (N=2):**
- P1 = Morpho Blue (PRS 1.87)
- P2 = Pendle (PRS 4.24, L3 strategy receipt issuer)
- Ethena (PRS 4.42) NOT counted — captured via sUSDe L2 dual-assessment per §4.2.2
- ProtocolRisk = max(1.87, 4.24) + 0.20 × mean(1.87, 4.24) = 4.24 + 0.61 = **4.85**

**AssetRisk** (USDC supply-side, single-asset):
- AssetRisk = USDC ARS 2.50

**S-factor scoring (Type 3 Mode A direct N=1, with v3.8 adjustments):**
- S1 base = 4 (Morpho Blue oracle + LLTV + IRM standard)
- **S1 v3.8 +1 recursive penalty = 5**
- S2 = mapping(sUSDe ARS 4.25) = 4 (no floor binding because already at 4)
- S3 = 5 (typical utilization)
- S4 = 7 (N=1 single-collateral floor + Pendle correlation risk)
- S5 = 4 (market youth)

SSR = 0.30·5 + 0.25·4 + 0.20·5 + 0.15·7 + 0.10·4 = 1.5 + 1.0 + 1.0 + 1.05 + 0.40 = **4.95**

**GRS = 0.35·4.85 + 0.25·2.50 + 0.40·4.95 = 1.70 + 0.625 + 1.98 = 4.31**

⇒ APPROVED (margin 3.19 to 7.5 hard cap).

### 4.3 Cap binding
- L1 Rule 2 (Morpho Blue 1.87, no limit)
- L1 Rule 2 (Pendle 4.24, band 4.1-5.5) → 30%
- L2 Rule 3 (USDC 2.50, no limit)
- L4 Rule 4 Type 3 S2=4 → 20%
- **Binding cap: 20% per vault** (Rule 4 Type 3)

---

## 5. Audit Pass on Existing Registry (2026-05-07)

### 5.1 Affected strategies (none currently in registry)

No L3 strategies in the current registry use Pendle PT, AMM LP, or non-L2 lending receipt tokens as collateral in lending markets. The v3.8 amendment is **forward-looking**.

### 5.2 Boundary cases (verified unaffected)

The following yield-bearing wrappers are L2 per §0.1 and use existing S2 rules (NOT triggered by v3.8):
- syrupUSDC, syrupUSDT (Maple)
- sUSDS (Sky)
- sUSDe (Ethena)
- earnAUSD-Monad (Upshift)
- pufETH (Puffer)
- msY (Mainstreet)

Existing strategies using these (Morpho msY/USDC, Morpho earnAUSD/USDC Monad, etc.) continue to use the standard §6.2 S2 mapping with no changes.

### 5.3 No verdict changes from v3.8

Zero registry impact at activation. v3.8 establishes the framework for future strategy assessments.

---

## 6. Conservative-by-design rationale

The v3.8 amendment maintains the methodology's "fail conservatively" principle:

1. **N+1 protocol counting** captures composition risk explicitly (vs. ignoring Pendle/Yearn protocol risk in current methodology)
2. **S2 floor 4** prevents understating recursive complexity even with high-quality underlying
3. **S1 +1 penalty** captures oracle/mechanism risk specific to wrapped strategy collateral
4. **Cascade-EXCLUDE** if underlying L3 is excluded ensures no path-around-exclusion via market composition
5. **No floor relaxation** — the existing v3.7 hard caps still apply at L3 (GRS > 7.5 EXCLUDED)

The rule is **forward-looking** — does not retroactively change existing assessments — and operationally aligned with how risk frameworks in the broader industry (LlamaRisk, Gauntlet) treat composition.

---

## 7. Implementation checklist

- [x] `methodology/v38_amendment.md` — this document
- [ ] `methodology/risk_methodology.md` §4.2.2 — add v3.8 footnote about strategy-receipt collateral counting
- [ ] `methodology/risk_methodology.md` §4.3.5 / §6.2 — add Recursive Strategy Collateral Rule reference
- [ ] `methodology/layer3_strategy_assessment_methodology.md` §6.2 — add subsection on S2 + S1 adjustments
- [ ] `methodology/risk_methodology.md` §7 Changelog — add v3.8 entry
- [ ] Update version headers v3.7 → v3.8

---

## 8. Approval

This amendment was authored by the Risk Analyst Agent under CEO direction, date 2026-05-07. The amendment was triggered by an empirical observation during the v3.7 audit of Type 3 lending strategies: the methodology lacked an explicit rule for composition cases (Pendle PT collateral, LP collateral, lending receipt collateral), creating undefined behavior for assessments that should be possible.

**Conservative-by-design properties preserved:**
- Tail-risk pricing maintained (S1 +1, S2 floor 4)
- No relaxation of existing hard caps
- N+1 protocol counting strengthens the Multi-Protocol Rule rather than weakening it
- Cascade rules preserve the L1/L2 cascade integrity

Approved for v3.8 release: 2026-05-07.
