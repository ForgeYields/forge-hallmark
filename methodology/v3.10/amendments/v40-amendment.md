# ForgeYields Risk Methodology — v4.0 Amendment

**Date:** 2026-05-18
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** Introduction of Layer 0 (Chain Risk) + chain-as-dependency integration into existing Multi-Protocol Rule
**Backwards compatibility:**
- Layers 1, 2, 3 unchanged in their internal rubrics
- GRS formula unchanged — chain risk enters via the existing `protocol_risk` component through extended dependencies
- Strategies assessed under v3.x retain their GRS until naturally re-scored

---

## 1. Overview

v4.0 closes a methodology gap exposed by the deployment of strategies on novel chains (Monad, Hyperliquid HyperEVM, Plasma): the framework pre-v4.0 had no explicit way to score **chain-level risk** as distinct from protocol-level smart-contract risk. Cross-chain bridge risk was partially captured via L1 C5 (v3.6 amendment), but the foundational risk of operating on an immature or centralized chain itself had no dedicated layer.

**Empirical observation:** Morpho Blue on Ethereum (PRS 1.87, $2B+ TVL, 5-year operational history) and Morpho Blue on Monad (same code, $10M TVL, 6 months old, novel parallel-execution VM) are treated identically by the v3.x methodology — same `morpho-blue.yaml` produces the same `protocol_risk` component regardless of chain. This materially under-rates strategies on newer/centralized chains.

**Hyperliquid HyperEVM precedent:** Already assessed as L1 (`category: l1-chain`) under v3.10 as a one-off — v4.0 formalizes this pattern into a dedicated layer with its own rubric and a clean integration mechanism.

**v4.0 resolution:** Introduce **Layer 0 — Chain Risk Score (CRS)** with chain-specific N1–N5 criteria, and integrate chains into the existing **Multi-Protocol Rule** as additional dependencies. No new GRS formula, no new arbitrary weight constant — the chain risk compounds with protocol risk naturally via the rule that already governs stacked protocol dependencies.

---

## 2. Motivation — why a dedicated chain layer

### 2.1 Conceptual distinction

| Dimension | Smart-contract protocol (L1) | Chain (L0) |
|---|---|---|
| What it is | Application logic deployed on a chain | The execution environment itself |
| Failure modes | Bugs, governance attacks, oracle failures | Consensus halt, sequencer compromise, validator collusion, VM bugs, deep reorgs |
| Code audit scope | Application contracts (Solidity/Vyper/Cairo) | VM, consensus, sequencer, bridge contracts |
| Maturity signal | TVL, operational age of the application | Operational age of the chain, validator decentralization |
| Recovery from failure | Pause + redeploy | Hard fork, chain restart, irreversible loss |

Mixing these in the same L1 layer with the same C1–C6 rubric is a category error. A "C1 Audit Status" question for Morpho Blue (3 audits by Spearbit, OpenZeppelin, ChainSecurity) is fundamentally different from "C1 Audit Status" for the Monad VM (parallel execution engine, MonadBFT consensus — different attack surface entirely).

### 2.2 Per-chain protocol re-deployment is **not** a new protocol

A user might suggest: "score Morpho-Ethereum and Morpho-Monad as separate L1 entries". v4.0 rejects this approach because:

1. **Same code = same smart-contract risk**. Re-deploying the same audited contract to a new chain does not change C1 (audits), C3 (governance), C5 (smart contract risk), or C6 (team). These remain protocol properties.
2. **Per-chain forking would explode the registry**. Morpho is on ~10 chains; Aave V3 on 13+. The maintenance burden is unsustainable.
3. **The actual chain-level risk** (consensus age, sequencer setup, VM novelty) is **shared across all protocols on that chain**, so it belongs in a separate layer that strategies can reference.

→ One L1 protocol entry per protocol (global, multi-chain). Chain-specific risk lives in L0.

### 2.3 The v3.6 cross-chain rule does not solve this

v3.6 (`C5 Cross-Chain Configuration Hard Cap`) addresses **the protocol's bridge dependency** — e.g., Renzo using LayerZero OFT with 1-of-1 DVN config. This is a property of the protocol's design, not of the chain it sits on. It does not capture:
- Consensus risk of the chain itself
- Sequencer halt history
- VM-level bugs
- Validator centralization

v4.0 adds the missing layer; v3.6 remains in force for the cross-chain rule it defines.

---

## 3. Layer 0 — Chain Risk Score (CRS)

### 3.1 Definition

The **Chain Risk Score (CRS)** is a 1–10 score for an individual blockchain (L1, L2, or L3 chain). It represents the cumulative risk that arises from operating on this chain, independent of any specific protocol or asset.

CRS is published per chain in `forge-hallmark/scores/chains/{slug}.yaml`.

### 3.2 N-criteria rubric (Network criteria)

CRS is computed as a weighted average of five **N-criteria** (N for "Network"):

| Criterion | Weight | What it measures |
|---|---|---|
| **N1 Consensus & Decentralization** | 25% | Validator count, BFT threshold, finality model, validator selection criteria, slashing mechanisms |
| **N2 Sequencer Architecture** | 20% | Sequencer centralization (single vs distributed vs proof-based), bondedness, censorship resistance, fallback mechanisms |
| **N3 Operational Track Record** | 20% | Operational age, uptime history, sequencer halts, deep reorgs, recovered incidents |
| **N4 Bridge Architecture** | 15% | Canonical bridge security (multisig threshold, audit coverage, attack surface), native asset bridging mechanism |
| **N5 VM Maturity** | 20% | VM novelty (EVM-equiv vs custom), audit coverage of runtime, formal verification, known bug history |

**Weighted formula:**
```
CRS = 0.25·N1 + 0.20·N2 + 0.20·N3 + 0.15·N4 + 0.20·N5
```

**Scoring scale**: same 1–10 convention as PRS/ARS (1 = lowest risk, 10 = highest).

### 3.3 N1 — Consensus & Decentralization

| Band | Description |
|---|---|
| 1–2 | Highly decentralized PoS (Ethereum: 800K+ validators), economic finality, slashing active, multi-client diversity |
| 3–4 | Mature L2 with proof system (Arbitrum, Optimism, Base) or PoS chain with 100+ validators + slashing |
| 5–6 | Smaller PoS / DPoS with 20–100 validators OR zk-L2 with prover decentralization in progress (Starknet, zkSync) |
| 7–8 | Small validator set (<20) OR single-validator/sequencer chain OR consensus security unproven at scale |
| 9–10 | Single validator (closed-source consensus, sequencer-only), known consensus bugs, no slashing |

### 3.4 N2 — Sequencer Architecture

| Band | Description |
|---|---|
| 1–2 | Fully decentralized sequencer OR Ethereum L1 (no sequencer concept) |
| 3–4 | Centralized sequencer with on-chain force-exit + open sequencer roadmap (Arbitrum, Optimism, Base) |
| 5–6 | Centralized sequencer without force-exit OR shared sequencer in early phase |
| 7–8 | Centralized sequencer with documented halt history OR closed-source sequencer |
| 9–10 | Centralized sequencer with no recovery mechanism + closed-source + halt history without remediation (Hyperliquid pre-2026) |

### 3.5 N3 — Operational Track Record

| Band | Description |
|---|---|
| 1–2 | >5 years mainnet operational, zero sustained outages, clean recovery from any incident |
| 3–4 | 2–5 years operational, no sustained outages, mature operations |
| 5–6 | 1–2 years operational OR 2+ years with documented outages well-recovered |
| 7–8 | <1 year operational OR 1+ year with sustained halt (>4 hours) without strong remediation |
| 9–10 | <6 months operational OR multiple sustained halts OR unrecovered loss-causing incidents |

### 3.6 N4 — Bridge Architecture

| Band | Description |
|---|---|
| 1–2 | Native L1 (Ethereum — no bridge needed) OR canonical bridge with strong multisig (≥7-of-10 + timelock) + multiple audits |
| 3–4 | Canonical bridge multisig 5-of-9 to 7-of-10, audits public, no incidents |
| 5–6 | Canonical bridge multisig 3-of-5 OR 4-of-7 with limited audit coverage |
| 7–8 | Smaller bridge multisig (<3-of-N) OR undisclosed threshold OR documented bridge incident |
| 9–10 | Single-key bridge OR 1-of-N multisig OR irreversible bridge loss incident |

### 3.7 N5 — VM Maturity

| Band | Description |
|---|---|
| 1–2 | EVM-equivalent or EVM-mature (5+ years), well-audited runtime, no critical bugs in recent past |
| 3–4 | EVM-compatible mature L2 with minor divergences, runtime audited |
| 5–6 | EVM-compatible but with novel features (parallel execution, custom precompiles) <2 years live |
| 7–8 | Novel VM (Cairo, MoveVM, MonadVM) with limited audit coverage OR EVM with known unpatched bugs |
| 9–10 | Novel VM <6 months mainnet + no public audit + known bug surface |

### 3.8 Hard caps

Same convention as PRS:
- **CRS > 7.5** → EXCLUDED (chain too risky for any strategy)
- **Any single N-criterion ≥ 9** → mandatory review (likely hard cap depending on criterion)

---

## 4. Integration into L3 GRS — chain as dependency

### 4.1 No new formula

The existing L3 GRS formula is preserved unchanged:

```
GRS = w_PR · protocol_risk + w_AR · asset_risk + w_SSR · strategy_specific_risk
```

Chain risk enters via the existing `protocol_risk` component, using the **Multi-Protocol Rule already in force since v3.1**.

### 4.2 Extended Multi-Protocol Rule

Pre-v4.0:
```
protocol_risk = MPR([PRS_protocol1, PRS_protocol2, ...])
              = max(PRSs) + (N-1) · 0.20 · mean(PRSs)
```

v4.0:
```
protocol_risk = MPR([CRS_chain1, CRS_chain2, ..., PRS_protocol1, PRS_protocol2, ...])
              = max(all_scores) + (N_total-1) · 0.20 · mean(all_scores)
```

The formula is identical; the input list is extended to include chain scores. Chains are conceptually platform protocols that the strategy depends on, so they belong in the dependencies list.

### 4.3 Why this is methodologically defensible

1. **Zero new constants.** No new weight, no new offset, no new scaling factor. The MPR is already justified in v3.1 ("stacked protocol dependencies compound attack surface and operational complexity"). That rationale applies identically to chains.

2. **Semantic consistency.** A strategy depends on its chain in the same way it depends on its protocols — both are external systems the strategy needs to function correctly. Excluding chains from the dependency list was an oversight.

3. **No double-counting.** L1 C5 (cross-chain config) captures the **protocol's bridge to other chains**. L0 CRS captures the **chain itself**. These are distinct concerns and do not overlap.

4. **Backwards-compatible math.** For Ethereum-only strategies, the CRS is low (~1.5) and very close to a typical protocol's PRS, so MPR inflation is minimal. The new component is most visible for strategies on high-CRS chains (Monad, Hyperliquid) — exactly where we want it visible.

### 4.4 Worked example

**Strategy: Morpho Blue earnAUSD/USDC on Monad**
- Dependencies (v3.x): protocols = [morpho-blue PRS 1.87]
  - protocol_risk = MPR([1.87]) = 1.87
- Dependencies (v4.0): chains = [monad CRS 5.5], protocols = [morpho-blue PRS 1.87]
  - protocol_risk = MPR([5.5, 1.87]) = max(5.5, 1.87) + (2-1) × 0.20 × mean(5.5, 1.87)
                 = 5.5 + 0.20 × 3.685 = **6.24**

The chain risk is now captured. The strategy's GRS is recomputed using protocol_risk=6.24 instead of 1.87, producing a more honest risk assessment.

**Strategy: eUSDC-95 on Ethereum**
- Dependencies (v3.x): protocols = [euler PRS 2.93]
  - protocol_risk = MPR([2.93]) = 2.93
- Dependencies (v4.0): chains = [ethereum CRS 1.5], protocols = [euler PRS 2.93]
  - protocol_risk = MPR([1.5, 2.93]) = max(1.5, 2.93) + 0.20 × mean(1.5, 2.93)
                 = 2.93 + 0.20 × 2.215 = **3.37**

Small bump (+0.44) reflecting that even Ethereum is not zero risk. The strategy's GRS goes up slightly but does not change verdict.

### 4.5 Multi-Chain support (deferred)

For strategies operating on multiple chains simultaneously (rare today), the same MPR applies — multiple chain entries in `dependencies.chains` are included alongside protocol entries. No separate "Multi-Chain Rule" is required.

For the current registry (all 47 strategies are single-chain), this case is theoretical. The schema supports it; the math handles it.

---

## 5. Schema changes

### 5.1 New: `chains/{slug}.yaml`

```yaml
# yaml-language-server: $schema=../../schemas/chain.schema.json
name: Monad
slug: monad
category: l1-chain  # or l2-rollup, l2-validium, etc.
vm_type: monad-evm  # or evm, cairo, move, etc.
assessment:
  date: '2026-05-18'
  methodology_version: v4.0
crs: 5.5
criteria:
  n1: 6  # Consensus & Decentralization
  n2: 7  # Sequencer Architecture
  n3: 8  # Operational Track Record
  n4: 5  # Bridge Architecture
  n5: 6  # VM Maturity
```

### 5.2 Updated: `strategy.schema.json`

Add optional `dependencies.chains: string[]`:

```yaml
dependencies:
  chains: [monad]               # NEW in v4.0 (optional, defaults to [])
  protocols: [morpho-blue]
  assets: [ausd-monad, usdc]
```

### 5.3 Migration of `hyperliquid-hyperevm.yaml`

The Hyperliquid HyperEVM entry currently in `public/scores/protocols/` with `category: l1-chain` is migrated to `public/scores/chains/hyperevm.yaml` with the new chain schema and N1–N5 rubric (re-assessment under v4.0). The old entry is removed.

---

## 6. Validation rules (Hallmark scripts)

`scripts/validate-scores.js`:
- Add `chains` layer (loaded from `public/scores/chains/`)
- Validate against `chain.schema.json`

`scripts/check-cascade-integrity.js`:
- Verify every strategy's `dependencies.chains[]` slug resolves to an existing chain YAML

`scripts/check-drift.js`:
- Add chains layer with weights `{n1:0.25, n2:0.20, n3:0.20, n4:0.15, n5:0.20}`
- Enforce `crs == weighted_sum(n_criteria)` within ±0.05

---

## 7. Migration path

### 7.1 New assessments

All L3 strategy assessments completed under v4.0 onwards MUST include `dependencies.chains` and recompute `protocol_risk` using the extended MPR.

### 7.2 Existing assessments (v3.x)

Existing strategy YAMLs (assessed under v3.4, v3.6, v3.7, v3.10) retain their current GRS until naturally re-scored. The `assessment.methodology_version` field documents which version was used. The frontend already does not display this version per recent UI cleanup (2026-05-18) — internal audit trail only.

When a v3.x strategy is re-scored, it is upgraded to v4.0 and gains the `dependencies.chains` field.

### 7.3 Priority re-scoring queue

Strategies that benefit MOST from v4.0 re-scoring (chain risk currently mis-priced):
1. earnAUSD/USDC (Monad) — high-CRS chain
2. morpho-blue-wsteth-weth-looping (Monad)
3. MorphoBlue-kHYPE-USDC-77 (HyperEVM)
4. MorphoBlue-kHYPE-USDT0-77 (HyperEVM)

Strategies that change minimally (mature chain):
- All Ethereum strategies (eUSDC-95, PT-USDat, etc.) — small +0.3 to +0.5 bump on protocol_risk, negligible GRS impact

---

## 8. Changelog (to append to `risk_methodology.md` §7)

```
| 4.0 | 2026-05-18 | NEW LAYER — Layer 0 Chain Risk Score (CRS).
Introduces dedicated chain-level rubric (N1 Consensus, N2 Sequencer, N3
Operational Track Record, N4 Bridge Architecture, N5 VM Maturity).
Chains integrate into L3 GRS via the existing Multi-Protocol Rule —
no new formula, no new weight constant. Chain entries published in
public/scores/chains/{slug}.yaml. Hyperliquid HyperEVM migrated from
protocols/ to chains/. Initial L0 batch: ethereum, arbitrum, base,
starknet, monad, hyperevm. See methodology/v40_amendment.md. |
```

---

## 9. Open questions for future amendments

- **CRS time decay** (analog of v3.5 C4 max rule): should chain operational age automatically improve N3 over time without re-assessment? (Likely yes, with quarterly review).
- **Cross-chain strategy risk floor**: for strategies actively bridging assets between chains, should `chain_risk` have a floor of the bridge-protocol's own audit/security risk? (Likely captured already via L1 C5.)
- **L0-L1 cascade rule**: if a chain becomes EXCLUDED (CRS > 7.5), should all strategies on that chain auto-EXCLUDE? Probably yes — analog of v3.x cascade exclusion rules.
