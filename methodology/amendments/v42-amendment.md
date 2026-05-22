# ForgeYields Risk Methodology — v4.2 Amendment

**Date:** 2026-05-22
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** L3 procedural clarification — separate **GRS computation** from **verdict derivation** in cascade-exclusion scenarios.
**Backwards compatibility:**
- No change to any rubric, criterion, weight, threshold, or formula
- No change to eligibility cutoffs (GRS ≤ 7.5 unchanged, hard caps unchanged)
- No retroactive impact on existing scores or verdicts
- Pure procedural clarification: aligns methodology text with the implementation reality already in production score files

---

## 1. Overview

Pre-v4.2 methodology text in three locations instructed assessors to **skip computing S-factors (or X-factors) when a cascade hard cap fired**:

> *"Cascade rule: If underlying L2 ARS > 6.5, strategy is EXCLUDED via §0.2.2. **Do not compute S2; document the cascade.**"*

(`layer3_strategy_assessment_methodology.md` §342, §401, and `full-framework.md` §629)

This instruction is at odds with the implementation reality. Live score files in `forge-hallmark/scores/strategies/` carry **fully-computed GRS values even when cascade-excluded**. Concrete example: `pendle-pt-apyusd-18jun2026.yaml` carries `grs: 7.14` with all components and S-criteria populated, despite its dependency `apyusd` having `ars: 6.6` — above the v3.7 L2 hard cap and therefore a cascade trigger.

v4.2 formalizes this separation: **GRS is always computed; the verdict is a separate field derived from both the GRS and any cascade triggers. In conflict, the verdict prevails.**

---

## 2. The procedural change

### 2.1 GRS is always computed in full

For every assessed strategy, regardless of any cascade triggers:

- All component scores (PR, AR, SSR) are computed per the standard rubrics
- All S-criteria (or X-criteria for Type W) are scored and populated
- The composite GRS is computed via the standard formula `GRS = 0.35·PR + 0.25·AR + 0.40·SSR` and published
- No field in the YAML score record is left empty due to cascade exclusion

### 2.2 Verdict is derived separately

The strategy's `verdict` field (or equivalent eligibility flag in the registry) is a **separate output** from GRS. It is determined by the following decision tree:

```
1. Any L1 hard cap triggered on a participating protocol?   → verdict: excluded
2. Any L2 hard cap triggered on a participating asset?      → verdict: excluded
3. Any Type W X-hard cap triggered (where applicable)?      → verdict: excluded
4. Composite GRS > 7.5?                                     → verdict: excluded
5. Composite GRS in 6.0–6.5 WATCHLIST band                  → verdict: watchlist
6. Otherwise                                                → verdict: approved
```

Any of steps 1–4 firing produces `verdict: excluded` regardless of the GRS value. A GRS of 7.14 with a triggered cascade is `verdict: excluded`, not "approved with GRS under 7.5".

### 2.3 Verdict prevails over GRS for deployment

The allocator and any downstream consumer **MUST** read the `verdict` field for deployment decisions, not the raw GRS. A strategy with `verdict: excluded` is non-deployable irrespective of how low its GRS is.

The `grs` field exists for **continuous measurement** (see §3), not for eligibility gating.

---

## 3. Why preserve the full GRS

Continuing to compute GRS for cascade-excluded strategies serves four purposes:

1. **Continuous measurement.** "How far does this strategy miss?" is meaningful even for excluded strategies. A cascade-excluded strategy with a GRS of 4.0 (good standalone score, bad dependency) is qualitatively different from one with GRS 8.5 (bad on all axes).

2. **Time-series tracking.** When an L2 asset's ARS drops back below the hard cap (e.g., a depeg recovers and A2 is re-scored), the strategy's previously-computed GRS allows immediate verdict re-evaluation without a fresh assessment cycle.

3. **Immediate reactivation when cascades lift.** If the dependency that triggered the cascade is rescored to clearing, the strategy's already-published GRS supports same-cycle re-enabling. No "we need to re-score from scratch" delay.

4. **No empty fields in the registry.** Every strategy record carries a complete, defensible GRS payload. Consumers (allocator, frontend, integrators) never have to handle null components.

---

## 4. Text replacements (four locations)

No threshold, weight, rubric, or criterion is touched — pure procedural language. Four exact before/after pairs:

### 4.1 `layer3_strategy_assessment_methodology.md` §342 (Type 2B Pendle LP — S2 cascade)

**Before:**
> Cascade rule: If underlying L2 ARS > 6.5, strategy is EXCLUDED via §0.2.2. Do not compute S2; document the cascade.

**After:**
> **Cascade rule:** If underlying L2 ARS > 6.5 (v3.7 L2 hard cap), the strategy verdict is forced to **EXCLUDED** via §0.2.2 regardless of composite GRS. Per v4.2, S2 (and all other components / criteria / composite GRS) is still computed and published — the verdict field carries the cascade exclusion; the GRS provides continuous measurement of how the strategy would score if the cascade lifted. Assets in WATCHLIST band (ARS 6.0–6.5) trigger §5 Rule 3 concentration cap, not exclusion.

### 4.2 `layer3_strategy_assessment_methodology.md` §401 (Type 2C Pendle PT — S1 cascade)

Same wording as §4.1 with `S1` substituted for `S2`.

### 4.3 `layer3_strategy_assessment_methodology.md` §74–§75 (cascade-excluded report structure)

**Before:**
```
If cascade excludes the strategy, the assessment output format is truncated to:
- Summary header
- §0 (classification + cascade table showing which trigger fired)
- §9 (GRS computation skipped, verdict EXCLUDED documented)
- Omit §5–§8 S-factor scoring (non-informative)
```

**After:**
```
If cascade excludes the strategy, the assessment output remains fully computed
(per v4.2: GRS is always derived; verdict is a separate field). Required sections:
- Summary header
- §0 (classification + cascade table showing which trigger fired)
- §5–§8 S-factor scoring completed in full (supports continuous measurement and
  same-cycle reactivation if the cascade lifts — see v4.2 §3)
- §9 documents the full GRS computation AND the verdict: excluded derived from
  the cascade trigger (verdict prevails for deployment per v4.2 §2.3)
```

### 4.4 `full-framework.md` §629

Same pattern as §4.1, with `§4.2.1` referenced instead of `§0.2.2`.

---

## 5. Acceptance criteria

After v4.2 is applied:

- ✅ All four text locations (§74–75, §342, §401 in L3; §629 in full-framework) match the new wording
- ✅ No occurrence of "GRS computation skipped", "Do not compute S1/S2", or "Omit §5–§8" remains anywhere in `methodology/`
- ✅ No threshold, weight, criterion, or rubric is modified
- ✅ No existing score record requires recomputation or republication
- ✅ Validators (`validate-scores.js`, `check-cascade-integrity.js`) continue to pass without modification — they already enforce the "GRS always present" invariant in practice
- ✅ Downstream consumers (allocator, frontend, API) continue reading the verdict field for eligibility; no contract changes required

---

## 6. Non-changes (explicit)

- The v3.7 L2 hard cap threshold (ARS > 6.5) is unchanged
- The 6.0–6.5 WATCHLIST band semantics are unchanged
- The §0.2.3 Weakest Link Rule is unchanged
- All S/X criterion rubrics are unchanged
- The GRS composition formula `0.35·PR + 0.25·AR + 0.40·SSR` is unchanged
- The eligibility cutoff GRS ≤ 7.5 is unchanged
- All existing assessments remain valid as written
