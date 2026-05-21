# ForgeYields Risk Methodology — v3.9 Amendment

**Date:** 2026-05-07
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** Layer 1 §C3 (Governance Quality) — formalize custody mode recognition for drain-vector role authority
**Backwards compatibility:** v3.7/v3.8 unchanged; only L1 C3 rubric gets a new "Custody Mode Recognition" sub-rule

---

## 1. Overview

v3.9 closes a methodology gap identified during May 2026 stablecoin issuer assessments (Saturn, Noon, Reservoir): the methodology pre-v3.9 treats every "single EOA on drain-vector role" as C3=9 EXCLUDED regardless of off-chain custody mode (plain hot wallet, HSM, AWS-KMS, MPC threshold-signature).

**Empirical observation:** During Saturn's response to its EXCLUDED verdict, the team disclosed that the deployer EOA is actually a Fireblocks MPC wallet with 2/3 approval policy outside the initiator. This raised the question: does MPC threshold-signature meaningfully change the C3 score, given that on-chain output is indistinguishable from a plain EOA?

**v3.9 resolution:** Introduce explicit Custody Mode Recognition with conservative-by-design defaults:
- Plain EOA / HSM-single-key remain C3=9 EXCLUDED (UNCHANGED from prior methodology)
- MPC threshold-signature **with verifiable attestation** can mitigate to C3=7-8 (qualitative)
- **Only on-chain Timelock + multisig** can fully clear C3 to bands 4-6

**Net registry impact at activation:** Pending attestation collection from current EXCLUDED protocols (Saturn, Noon, Reservoir). Forward-looking framework formalized.

---

## 2. Motivation — the verifiability problem

### 2.1 On-chain output identity

Three different custody modes produce **identical on-chain footprint**:

| Custody mode | `eth_getCode` | Signature format | Policy enforcement |
|--------------|---------------|------------------|---------------------|
| Plain hot wallet | `0x` | ECDSA via single key | None (single device) |
| HSM / AWS-KMS-backed | `0x` | ECDSA via cloud HSM | API-policy via cloud |
| Fireblocks MPC 2/3 | `0x` | ECDSA via threshold-aggregation | Off-chain Policy Engine |

⇒ **Auditors cannot distinguish these three modes from on-chain reads alone.** This was the methodology's blind spot pre-v3.9.

### 2.2 Security model differences

| Attack vector | Plain EOA | HSM single-key | MPC 2/3 (threshold) |
|---------------|-----------|----------------|---------------------|
| Single device compromise | drain | N/A | mitigated (need 2 of 3 systems) |
| Cloud zero-day | N/A | drain (Resolv pattern) | mitigated (multi-cloud) |
| Insider single-party | drain | drain | mitigated (need 2 colluding) |
| Custodian compromise (e.g., Fireblocks supply-chain) | N/A | N/A | partial drain risk |
| Customer config error (off-chain) | N/A | N/A | drain (policy bypass) |

⇒ **MPC 2/3+ is materially safer than plain EOA / HSM** for drain-vector authority, BUT depends on custodian + customer config integrity that cannot be verified on-chain.

### 2.3 The asymmetric protection problem

Pre-v3.9 strict reading: any single on-chain EOA on drain-vector → C3=9 EXCLUDED. This correctly blocks plain EOAs and HSM patterns (Resolv) but **also blocks legitimate MPC-managed addresses**, creating a false-positive that may exclude well-secured institutional protocols.

**Saturn case study:** $64.5M TVL protocol with doxxed UPenn-alumni team, 3 audits (Three Sigma + 2 Certora), institutional VC backing — would have been APPROVED at PRS ~5.5 with multisig + timelock. The strict reading penalizes the disclosure gap, not the actual security.

---

## 3. Amendment — Custody Mode Recognition

### 3.1 Scoring matrix (drain-vector role authority)

```
═══════════════════════════════════════════════════════════════════
TIER A — Single Trust Domain (HARD CAP UNCHANGED, C3=9 EXCLUDED)
═══════════════════════════════════════════════════════════════════
  • Single plain EOA (any custody):                C3=9
  • HSM/AWS-KMS-backed single key:                 C3=9 (Resolv)
  • Single hardware wallet on drain-vector role:   C3=9
  
  Rationale: single point of failure, non-recoverable.

═══════════════════════════════════════════════════════════════════
TIER B — Multi-Trust-Domain Off-Chain (REQUIRES ATTESTATION)
═══════════════════════════════════════════════════════════════════
  • MPC threshold ≥2/3 with NO attestation:        C3=9 (default — strict)
  • MPC threshold ≥2/3 + signed attestation:       C3=7-8 (qualitative)
  • MPC threshold ≥3/5+ + attestation:             C3=6-7
  • MPC + 24h on-chain Timelock:                   C3=5-6
  
  Rationale: threshold-signing across multi-trust-domain mitigates 
  single-key compromise vector, but on-chain opacity prevents direct 
  verification. Custodian dependency (Fireblocks/Copper/BitGo Trust/
  Anchorage) becomes additional trust assumption.

═══════════════════════════════════════════════════════════════════
TIER C — On-Chain Verifiable (FULL MITIGATION POSSIBLE)
═══════════════════════════════════════════════════════════════════
  • On-chain Safe 2/3 (no timelock):               C3=6 (per expanded-C3)
  • On-chain Safe 3/5+ (no timelock):              C3=4-5
  • On-chain Safe + 24h Timelock:                  C3=2-4
  • Immutable contract (no upgrade authority):     C3=2-3
  
  Rationale: fully on-chain verifiable threshold + delay; no off-chain 
  trust assumptions required.
═══════════════════════════════════════════════════════════════════
```

### 3.2 Attestation requirements (Tier B)

For an MPC threshold-signature claim to receive C3 mitigation (move from Tier A=9 to Tier B=7-8), the protocol team must provide ALL of:

1. **Custodian attestation letter** — signed by the MPC custodian (Fireblocks, Copper, BitGo Trust, Anchorage, Coinbase Custody, or equivalent), confirming:
   - Specific Ethereum address(es) under MPC custody
   - Threshold scheme (e.g., 2/3, 3/5)
   - Custodian's tenant/workspace ID
   - Date of attestation (must be within 6 months)

2. **Policy Engine configuration** — screenshot or exported config showing:
   - Quorum members (anonymized OK)
   - Approval threshold
   - Initiator vs approver separation
   - Critical action policies (mint, burn, upgrade — different thresholds OK)

3. **Customer attestation** — signed by protocol team operations officer:
   - Confirms the MPC config has not been changed within last 30 days
   - Commits to public disclosure of any future config changes
   - Identifies the Compliance/Operations officer responsible

4. **Custodian quality tier** (qualitative scoring):
   - Tier 1 custodians (Fireblocks, Coinbase Custody, Anchorage, BitGo Trust): C3=7
   - Tier 2 custodians (Copper, BitGo, MPC.io, Hex Trust): C3=8
   - Unknown / smaller custodians: C3=9 (insufficient trust mitigation)

### 3.3 Verifiability requirements (Tier C)

For full on-chain mitigation, the drain-vector role must be transferred to:

1. **TimelockController** (or equivalent) verifiable on Etherscan with:
   - `getMinDelay()` ≥ 24h (preferably ≥48h for Tier 1 institutional)
   - Source code verified ("Source Code Verified, Exact Match")
   - Proposer role held by a multisig (Tier C-1 below) OR MPC-with-attestation (Tier B)

2. **Multisig (Gnosis Safe or equivalent)**:
   - On-chain readable owners + threshold via `getOwners()` + `getThreshold()`
   - Threshold ≥ 60% ratio AND ≥ 3 absolute parties (passes expanded-C3 dual-clause)
   - Public name attribution preferred (Etherscan tags, ENS) but not required

### 3.4 Hard cap rules (UNCHANGED from v3.6)

The following remain hard caps regardless of custody mode disclosure:

- **Single plain EOA on drain-vector** → C3=9 (default, no relaxation)
- **HSM/AWS-KMS-backed single key** → C3=9 (Resolv pattern, no relaxation)
- **MPC threshold without attestation** → C3=9 (default applies)
- **2-of-N or 3-of-N MULTISIG fails ratio AND absolute clauses** → C3=8 → EXCLUDED
- **C3 ≥ 8** → EXCLUDED (no exceptions)

### 3.5 Worked examples

**Example A — Saturn Protocol (current state, email disclosure only)**
- Email claim: Fireblocks MPC 2/3 outside initiator
- Attestation provided: NO (email only)
- → Tier A default applies: **C3=9 EXCLUDED stays**
- Recovery path: provide signed Fireblocks attestation + Policy Engine screenshot OR migrate to on-chain Timelock + Safe

**Example B — Saturn Protocol (after Fireblocks attestation)**
- Custodian: Fireblocks (Tier 1)
- Threshold: 2/3 + initiator separation
- Attestation: signed letter + Policy Engine config provided
- → Tier B applies: **C3=7** (mitigated)
- New PRS recompute: ~5.47 → **APPROVED, 30% cap**

**Example C — Saturn Protocol (after on-chain Timelock + Safe)**
- DEFAULT_ADMIN_ROLE transferred to TimelockController (24h delay)
- Timelock proposer = 3/5 Safe (on-chain verifiable)
- → Tier C applies: **C3=4-5** (full mitigation)
- New PRS recompute: ~4.87 → **APPROVED, 30% cap (potentially 50% if PRS<4.0)**

---

## 4. Audit Pass on Existing Registry (2026-05-07)

### 4.1 Affected protocols (current EXCLUDED state)

| Protocol | Pre-v3.9 status | Email/disclosure | Required attestation | v3.9 verdict |
|----------|-----------------|------------------|----------------------|--------------|
| **Saturn** | EXCLUDED PRS 5.87 | Fireblocks MPC 2/3 claimed via email | Fireblocks letter + Policy screenshot | **EXCLUDED stays** (email only insufficient); pending attestation |
| **Noon** | EXCLUDED PRS 6.33 | None | (custody method not yet disclosed) | **EXCLUDED stays** (default Tier A) |
| **Reservoir** | EXCLUDED PRS 6.08 | None | (custody method not yet disclosed) | **EXCLUDED stays** (default Tier A) |

### 4.2 Forward-looking application

Future L1 assessments where a protocol team discloses MPC custody:
- Strict reading default: C3=9 unless attestation provided
- With attestation: C3 score per Tier B matrix
- Re-assessment triggered by attestation receipt OR on-chain Timelock deployment

---

## 5. Outreach template for protocols seeking re-assessment

To accelerate re-assessment of EXCLUDED stablecoin issuers, the following outreach template should be sent:

```
Subject: Re-assessment opportunity — ForgeYields v3.9 methodology update

Dear [Protocol Team],

Following our [DATE] L1 assessment of [PROTOCOL] (verdict: EXCLUDED via 
C3=9 hard cap on single-EOA DEFAULT_ADMIN_ROLE), we've published a v3.9 
methodology amendment that formally recognizes MPC threshold-signature 
custody as a partial mitigation of the single-key risk pattern.

To trigger a re-assessment, please provide:

1. CUSTODIAN ATTESTATION LETTER signed by your MPC custodian (Fireblocks, 
   Copper, BitGo Trust, Anchorage, or equivalent), confirming:
   - Address [0x...] is under their MPC custody
   - Threshold scheme (e.g., 2/3, 3/5, with initiator separation)
   - Custodian's tenant/workspace ID
   - Attestation date (within 6 months)

2. POLICY ENGINE CONFIG showing:
   - Quorum members (anonymized OK)
   - Approval threshold
   - Initiator/approver separation
   - Critical action policies

3. CUSTOMER ATTESTATION signed by your operations officer confirming 
   no config changes within 30 days + commitment to public disclosure 
   of future changes.

ALTERNATIVELY (and preferred long-term path): migrate DEFAULT_ADMIN_ROLE 
to an on-chain Timelock + Safe configuration. This provides full on-chain 
verifiability and clears C3 to bands 4-6.

Once provided, we'll re-assess within 5 business days. Estimated outcome 
range based on disclosed custody:
- MPC attestation only: C3=7-8 → PRS ~5.4-5.7 → likely APPROVED at 30% cap
- On-chain Timelock + Safe: C3=4-5 → PRS ~4.9-5.1 → APPROVED at 30% cap

Best regards,
ForgeYields Risk Team
```

---

## 6. Conservative-by-design rationale

v3.9 maintains the methodology's core conservative principles:

1. **Default strict reading**: undisclosed custody → C3=9 (no relaxation without proof)
2. **Tier A unchanged**: plain EOA + HSM single-key still trigger hard cap (Resolv pattern protected)
3. **Off-chain attestation creates qualitative mitigation only**: MPC reduces to C3=7-8, never below 7 without on-chain Timelock
4. **On-chain proof remains gold standard**: full mitigation requires on-chain Timelock + Safe (Tier C)
5. **Custodian quality matters**: Tier 1 vs Tier 2 vs unknown distinguishes trust profile
6. **Attestation must be current**: 6-month staleness limit prevents historical letter misuse
7. **Policy change disclosure required**: customer attestation includes 30-day no-change clause

The amendment is **forward-looking** for active outreach but does NOT retroactively change the EXCLUDED status of Saturn/Noon/Reservoir without receipt of attestation.

---

## 7. Implementation checklist

- [x] `methodology/v39_amendment.md` — this document
- [ ] `methodology/risk_methodology.md` §2.3 C3 — add subsection on Custody Mode Recognition
- [ ] `methodology/risk_methodology.md` §7 Changelog — add v3.9 entry
- [ ] `methodology/layer1_protocol_assessment_methodology.md` C3 — add Tier matrix reference
- [ ] Update version headers v3.8 → v3.9
- [ ] Notion sync v3.9 amendment page
- [ ] Outreach to Saturn / Noon / Reservoir teams using §5 template

---

## 8. Approval

This amendment was authored by the Risk Analyst Agent under CEO direction, date 2026-05-07. Triggered by Saturn Protocol's response to their EXCLUDED verdict revealing Fireblocks MPC custody, which exposed a pre-existing methodology gap: inability to distinguish on-chain identical custody modes (plain EOA vs HSM vs MPC).

**Key design properties preserved:**
- Tail-risk pricing (Tier A unchanged for plain EOA / HSM)
- Strict reading default (no off-chain claims accepted without proof)
- Verifiability primacy (on-chain proof always preferred)
- Conservative tier ladder (no shortcut from C3=9 to C3<7 without on-chain Timelock)

**New capabilities enabled:**
- Re-assessment pathway for institutionally-secured protocols (Saturn pattern)
- Differentiation of custodian quality tiers (Tier 1 vs Tier 2)
- Outreach template to accelerate registry coverage
- Methodology readiness for upcoming MPC-managed institutional stablecoin protocols

Approved for v3.9 release: 2026-05-07.
