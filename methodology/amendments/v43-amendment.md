# ForgeYields Risk Methodology — v4.3 Amendment

**Date:** 2026-05-22
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** L1 C3 clarification — add a narrow carve-out for **regulated public-company issuers** to the v3.9 Custody Mode Recognition framework.
**Backwards compatibility:**
- No change to any rubric weight, formula, or eligibility cutoff
- No change to the v3.9 Tier A hard cap for unregulated entities (Resolv pattern stays EXCLUDED)
- No change to the multisig-threshold-floor rule
- Narrow, conditional carve-out applicable only to issuers meeting 4 cumulative pre-conditions
- Three currently-assessed protocols are eligible for re-classification under this amendment: Circle, Coinbase Custody Trust Company, BitGo Trust Company (post-IPO)

---

## 1. Overview

v3.9 (2026-05-07) introduced a hard cap on the C3 Governance criterion:

> *"Single EOA OR KMS-single-key on a critical drain-vector role (mint, burn, upgrade, price-setter, redemption-rule, blacklister) → C3 = 9 automatic trigger → EXCLUDED regardless of timelock."*

The amendment was written to catch the **Resolv pattern**: a small, unregulated issuer holding mint + price-setter roles on an AWS-KMS single-key. The hard cap correctly flags that exposure as catastrophic — a single-key compromise drains user funds with no recovery path.

However, the rule as written also catches **regulated public-company issuers** (Circle, Coinbase Custody, BitGo post-IPO) whose on-chain ProxyAdmin / Owner key topology looks identical to Resolv's, but whose operational control of that key is materially different. These issuers maintain SEC reporting obligations, SOC 1 Type 2 or SOC 2 Type 2 attestations explicitly covering on-chain key custody operations, banking-regulator supervision, and public disclosure of custody architecture. The drain-vector risk is functionally bounded by enterprise controls that do not exist in the Resolv case.

The result: v3.9 as written produces a **systemic false positive** that would force USDC, cbBTC, cbETH, and similar fiat-backed regulated stablecoins to EXCLUDED via cascade — disabling the entire fyUSDC vault and significant portions of fyETH/fyWBTC, against the substantive risk profile.

v4.3 introduces a narrow, conditional carve-out — a new **Tier B+ "Regulated Public Custody"** classification — that allows these specific issuers to be scored on the rest of the governance posture rather than auto-9.

---

## 2. The procedural change

### 2.1 Tier B+ — Regulated Public Custody

A protocol's on-chain governance keys may be classified as **Tier B+ (Regulated Public Custody)** — exempt from the v3.9 Tier A single-EOA hard cap — only if **ALL FOUR** of the following pre-conditions are satisfied cumulatively:

1. **Public company.** The issuer entity is a public company subject to SEC reporting obligations (10-K, 10-Q, 8-K, SOX §302/§404 ICFR) or equivalent jurisdiction (UK FCA listing, EU CSRD-regulated). Privately-held companies do not qualify regardless of size or revenue.

2. **Attested key custody.** A SOC 1 Type 2 OR SOC 2 Type 2 attestation, issued within the last 18 months by a Big-4 or recognized equivalent firm (BPM, Withum, Grant Thornton, Armanino, Wipfli), **explicitly covering the garde and operation of the on-chain keys that hold drain-vector authority over the assessed protocol** (ProxyAdmin, contract owner, upgrade authority, role-assignment authority). A generic SOC 2 over corporate IT infrastructure does NOT satisfy this condition — the attestation must specifically cover the on-chain key custody operations.

3. **Banking-regulator supervision.** The issuer entity holds an active license from a banking regulator: NYDFS BitLicense, OCC trust charter, FDIC-insured entity, or equivalent foreign-jurisdiction prudential regulator (FCA, BaFin, MAS, ASIC, etc.). Money-services-business registration alone does NOT satisfy this — must be banking-grade supervision with periodic examination authority.

4. **Public custody disclosure.** The issuer has filed a public attestation OR SEC filing OR regulator-published document establishing the existence of the custody control framework. The disclosure need not specify cryptographic key material, but must publicly establish that controls exist and are subject to ongoing review.

### 2.2 Effect of Tier B+ classification

If all four pre-conditions are satisfied, the v3.9 Tier A hard cap ("single EOA / KMS-single-key on drain-vector role → C3 = 9 automatic") is **suspended** for that protocol's L1 assessment. C3 is then derived from the rest of the governance posture (multisig structure on operational/role-assignment paths, timelock presence, signer doxx status, immutability of core contracts) per the standard rubric.

Tier B+ classification does NOT:
- Set a fixed C3 score (the analyst must still apply the rubric)
- Substitute regulatory oversight for technical mitigations on protocol upgrades (Timelock + multisig remain valued positively)
- Apply to any role other than the **on-chain governance keys controlling smart contract upgrades and role assignment**
- Apply to off-chain custody risk (which is a separate consideration captured under reserve risk in the asset's L2 assessment)

### 2.3 What Tier B+ does NOT change

- **v3.9 Tier A hard cap for unregulated entities** remains in full force. Any issuer not meeting all four pre-conditions is scored under the standard v3.9 rule.
- **Multisig-threshold-floor rule** (3-of-N with ratio + absolute thresholds) remains in full force and applies independently.
- **Multisig requires timelock without ≥4/N + ≥60% ratio rule** remains in full force.
- **DeFi protocols** (Aave, Morpho, Compound, etc.) are NOT eligible for Tier B+ — the carve-out applies only to fiat-backed regulated issuers with the four pre-conditions.

---

## 3. Why this is a narrow, defensible carve-out

### 3.1 The substantive risk distinction

The v3.9 amendment text reads: *"single-key compromise = single-point-of-failure regardless of delay window."*

This is correct for the Resolv pattern, where the "single key" is held under no external oversight. A malicious actor with key access has no human approval gate to bypass.

For regulated public issuers, the "single key" on-chain is operationally bounded by:
- Board of directors and audit committee oversight
- Internal audit function with regular review cycles
- SEC reporting obligations creating disclosure liability for material control failures
- SOC 1/2 Type 2 attestation requiring documented control evidence on a multi-month basis
- Banking-regulator examination authority with the power to revoke license
- Civil and criminal liability under securities law for material misrepresentations

A unilateral drain by a single Circle/Coinbase/BitGo employee with key access would trigger SEC enforcement, criminal prosecution, civil liability, and immediate license revocation. The drain-vector "single-point-of-failure" is bounded by a deep enterprise control stack that does not exist in the Resolv case.

This is not a defense of weak on-chain key topology in the abstract — it is recognition that the **same on-chain topology has different risk profiles depending on the off-chain control environment around it**.

### 3.2 What the SOC attestation specifically must cover

To distinguish a meaningful carve-out from a loophole, condition #2 (SOC attestation) requires the SOC to explicitly cover the **on-chain key custody operations**, not generic IT controls.

A SOC 2 Type 2 over a company's AWS environment, MFA policies, and access management is NOT sufficient.

A SOC 1 Type 2 (or SOC 2 Type 2) that names the on-chain key custody process — including key generation, segregation of duties, multi-party approval workflows, periodic key rotation, custodian relationships (Fireblocks/Anchorage/Coinbase Custody/etc.), and incident response — IS sufficient.

This distinction is critical. Without it, a protocol could obtain a generic SOC 2 and claim Tier B+ while the on-chain keys sit on a developer's laptop.

### 3.3 Eligibility under v4.3 (currently-assessed protocols)

| Protocol | Public co. | SOC covering on-chain keys | Banking regulator | Public disclosure | Qualifies? |
|----------|-----------|----------------------------|-------------------|-------------------|------------|
| **circle-internet-financial** | ✅ CRCL/NYSE (June 2025) | ✅ Deloitte SOC 1 Type 2, "Circle Mint key custody" | ✅ NYDFS BitLicense | ✅ 10-K + Reserve attestations | ✅ **Yes** |
| **coinbase-custody-trust-company** | ✅ COIN/NASDAQ | ✅ Deloitte SOC 1/2 Type 2, "key custody and storage operations" | ✅ NYDFS Limited Purpose Trust | ✅ 10-Q + custody attestations | ✅ **Yes** |
| **bitgo** | ✅ BTGO/NYSE (Jan 2026) | ⚠️ Verify Armanino SOC scope explicitly covers on-chain key operations | ✅ OCC National Trust Bank (Dec 2025) + state trust charters | ✅ 10-K + OCC examination | ⚠️ **Conditional** — pending SOC scope verification |
| **mainstreet-finance** | ❌ Privately held | ❌ No SOC | ❌ No banking regulator | ❌ — | ❌ **No** |
| **frax-finance** | ❌ Privately held | ❌ No SOC | ❌ — | ❌ — | ❌ **No** |
| **ipor-protocol** | ❌ — | ❌ — | ❌ — | ❌ — | ❌ **No** |
| **m-0-foundation** | ❌ — | ❌ — | ❌ — | ❌ — | ❌ **No** |
| **agora-finance** | ❌ — | ❌ — | ❌ — | ❌ — | ❌ **No** |
| All DeFi protocols | n/a (not eligible per §2.3) | — | — | — | ❌ **No** |

Two protocols qualify immediately; one (BitGo) requires verification of SOC scope before activating Tier B+.

---

## 4. Text replacement (single location)

`layer1_protocol_assessment_methodology.md` — §C3, immediately following the line 149 single-EOA hard cap clause.

**Insertion (new alinéa added after the line 149 Tier A clause):**

> **Tier B+ exception — Regulated Public Custody (v4.3, 2026-05-22):**
> The Tier A "single EOA / KMS-single-key on drain-vector role → C3 = 9" hard cap above is **suspended for issuers satisfying ALL FOUR cumulative pre-conditions** of v4.3 §2.1: (1) public company with SEC reporting (or equivalent), (2) SOC 1 Type 2 or SOC 2 Type 2 attestation within 18 months explicitly covering on-chain key custody operations (generic IT SOC does NOT qualify), (3) active banking-regulator license (NYDFS / OCC / FDIC / equivalent foreign), (4) public custody disclosure. Qualifying issuers are scored on the rest of the governance posture per the standard rubric.
>
> *Currently eligible: circle-internet-financial, coinbase-custody-trust-company. BitGo Trust Company pending SOC scope verification.*
> *Does NOT apply to: DeFi protocols, unregulated stablecoin issuers, or any entity failing one or more pre-conditions.*

No other text in `layer1_protocol_assessment_methodology.md`, `full-framework.md`, or any other file is modified by this amendment.

---

## 5. Acceptance criteria

After v4.3 is applied:

- ✅ The text insertion in layer1 §C3 (immediately after line 149) matches the wording in §4 above
- ✅ The v3.9 Tier A hard cap text on line 149 itself remains unchanged
- ✅ The multisig-threshold-floor rule remains unchanged
- ✅ No threshold, weight, criterion, or rubric is modified
- ✅ The amendments table in `methodology/README.md` and `public/README.md` carries a v4.3 row
- ✅ The `Current methodology version: v4.2` banner in headers is bumped to v4.3 (layer1, layer2, layer3, full-framework, risk_methodology, READMEs)
- ✅ Circle's `circle-internet-financial.yaml` is re-scored under v4.3: C3 9 → 7 (Tier B+ applied), PRS 4.40 → 4.00, verdict EXCLUDED → APPROVED with 50% per-vault cap
- ✅ Coinbase Custody's `coinbase-custody-trust-company.yaml` documents Tier B+ classification (no score change required — was already at C3=5 per pragmatic benchmark continuity; v4.3 ratifies this formally)
- ✅ BitGo's `bitgo.yaml` documents Tier B+ pending SOC scope verification as a follow-up item (no immediate score change — was already at C3=4 via on-chain Safe 8/13)
- ✅ Validators (`validate-scores.js`, `check-drift.js`, `check-cascade-integrity.js`) continue to pass without modification

---

## 6. Non-changes (explicit)

- v3.9 Tier A hard cap for unregulated entities is unchanged
- Multisig-threshold-floor rule (≥60% ratio OR ≥4 absolute + ≥40% ratio) is unchanged
- Timelock requirements for sub-floor multisigs are unchanged
- Emergency multisig handling (+1 penalty + cap at +2) is unchanged
- C3 weight (20% of PRS) is unchanged
- GRS formula `0.35·PR + 0.25·AR + 0.40·SSR` is unchanged
- Eligibility cutoff GRS ≤ 7.5 is unchanged
- All existing assessments for non-qualifying issuers remain valid as written
- DeFi protocols (Aave, Morpho, Compound, Curve, etc.) continue to be scored under standard v3.9 rule
- Off-chain reserve / fiat custody risk is unchanged (scored separately at L2 asset level)
