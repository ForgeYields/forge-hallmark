# ForgeYields Risk Methodology — v4.1 Amendment

**Date:** 2026-05-18
**Author:** Risk Analyst Agent (CEO-driven design)
**Scope:** L3 strategy types — introduce **Type W (Wrapper Vault)** for strategies that delegate execution to a third-party permissioned vault.
**Backwards compatibility:**
- v3.x, v4.0 unchanged for Types 1, 2A, 2B, 2C, 3
- Type W is a new addition with its own X1–X5 rubric
- GRS formula unchanged (PR × 0.35 + AR × 0.25 + SSR × 0.40); SSR for Type W = weighted sum of X1–X5

---

## 1. Overview

v4.1 closes a methodology gap exposed by Ipor Fusion vaults (fusnstETH, TAUCBETH, ipsrBTC) and Morpho MetaMorpho-style curated vaults (gteusdc-morpho): the pre-v4.1 framework forced classification into Type 1 (Looping), Type 2A/B/C (LP/PT), or Type 3 (Lending), none of which cleanly capture the **two-layer trust model** of vault-wrapped strategies.

**The conceptual issue:**

When ForgeYields directly executes a strategy (e.g. `spark-wsteth-weth-looping`), all risk lives at L1 (protocol) + L2 (asset) + L3 strategy-specific (S-factors describe the strategy execution).

When ForgeYields **deposits into a wrapper vault** that executes the strategy internally (e.g. `fusnstETH`), an additional trust layer exists:
- The vault's **Curator/Atomist** controls strategy parameters
- The vault's **exit mechanism** (queue/cooldown) gates depositor unwind
- The vault's **fee structure** affects depositor net yield
- The vault's **maturity** (age, TVL, incidents) is a depositor concern

The S-factors of Types 1/2/3 don't explicitly capture these wrapper-layer concerns. Pragmatic workarounds (e.g. broad reading of "S4 Unwind Liquidity" to include wrapper exit window) muddy the methodology.

**v4.1 resolution:** Introduce Type W with X1–X5 rubric that explicitly addresses wrapper-vault risk.

---

## 2. Type W — Wrapper Vault

### 2.1 Scope

Type W applies to any strategy where ForgeYields deposits into a third-party permissioned vault that internally executes a yield strategy. Examples:

- **Ipor Fusion vaults** (PlasmaVault with Atomist-controlled fuses)
- **Morpho MetaMorpho** (curator-controlled allocations across Morpho Blue markets)
- **Yearn V3 vaults** (strategist-controlled deployments)
- **Generic ERC-4626 wrappers** where a non-ForgeYields party operates the strategy

**Excluded from Type W** (classify under existing types instead):
- Direct deployments where ForgeYields controls execution (Spark/Aave loopers, direct Pendle PT/LP, direct Curve LP positions) — still Types 1/2/3
- Pure token wrappers without active strategy management (wstETH wrapping stETH, sUSDe wrapping USDe, etc.) — these are assets (L2), not strategies

### 2.2 X-criteria rubric (eXternal-vault criteria)

Type W replaces S1–S5 with X1–X5 in `strategy_specific_criteria`:

| Criterion | Weight | What it measures |
|---|---|---|
| **X1 Underlying Strategy Risk** | 25% | What the wrapper does economically (looping, LP, lending). Score as if assessed under the underlying type's S-factors, then take weighted avg. |
| **X2 Curator/Atomist Trust** | 30% | Who can move funds. v3.9 Custody Mode applies: plain EOA = 9, MPC-attested = 6–8, multisig + timelock = 3–5. |
| **X3 Exit Mechanism** | 20% | Depositor unwind path. Atomic = 1–2, queue ≤ 1d = 3–4, queue ≤ 7d = 5–6, queue > 7d or pause history = 7–9. |
| **X4 Fee Structure** | 15% | Mgmt + perf fees, hurdle rates, fee-change governance. Transparent low = 1–3, standard = 4–5, opaque/high = 6–8, unilateral change = 8–9. |
| **X5 Vault Maturity** | 10% | Age + TVL + incidents. >2y + >$100M = 1–3, >1y + >$10M = 4–5, <1y or <$10M = 6–8, <3 months or recent incident = 8–10. |

**Weighted formula:**
```
SSR_W = 0.25·X1 + 0.30·X2 + 0.20·X3 + 0.15·X4 + 0.10·X5
```

X2 gets the highest weight (30%) because curator/atomist trust is the **defining differentiator** of wrapper vaults vs direct strategies. A wrapped strategy with a single-EOA Atomist is fundamentally different from one with a multisig + timelock, even if everything else is identical.

### 2.3 X-criteria bands

#### X1 — Underlying Strategy Risk

The wrapper internally executes some strategy (looping, LP, lending, market-making, etc.). Score X1 as if you assessed that underlying activity under its native type, then take the **weighted average of its native S-factors**.

| Internal activity | How to score X1 |
|---|---|
| Wrapper does Type 1 (Looping) | Compute Type 1 S1–S4 for the internal strategy, take weighted avg |
| Wrapper does Type 3 (Lending allocation) | Compute Type 3 S1–S5 for the lending posture |
| Wrapper does Type 2A/B/C (LP) | Compute Type 2A/B/C S1–S4 |
| Wrapper does multi-strategy (e.g. MetaMorpho across N markets) | Allocation-weighted avg of per-market type scores |

#### X2 — Curator/Atomist Trust

| Band | Pattern |
|---|---|
| 1–2 | Multisig ≥ 4-of-N **AND** on-chain timelock ≥ 24h on drain-vector ops |
| 3–4 | Multisig ≥ 3-of-N with disclosed signers, no timelock OR timelock < 24h |
| 5–6 | Reputable curator entity (institutional name like Gauntlet, Steakhouse, Block Analitica) operating under documented framework, even if technically EOA-controlled |
| 7 | MPC-attested EOA via Tier-1 custodian (per v3.9 Tier B) |
| 8 | Undisclosed multisig threshold OR small multisig (≤ 2-of-N) |
| 9 | Plain EOA with no MPC attestation, drain-vector authority |
| 10 | Multiple EOAs sharing drain-vector authority, all unattested |

**Hard cap link to v3.9:** X2 = 9 (single plain EOA on drain-vector) triggers v3.9 Tier A → likely L3 EXCLUDED via S-factor cascade unless mitigated.

#### X3 — Exit Mechanism

| Band | Pattern |
|---|---|
| 1–2 | Atomic redeem (no queue, instant settlement against vault assets) |
| 3–4 | Queue with SLA ≤ 1 day documented, on-chain enforceable, no pause history |
| 5–6 | Queue with SLA ≤ 7 days, on-chain enforceable, ≤ 1 pause in last 12 months |
| 7–8 | Queue with SLA > 7 days OR queue is curator-discretionary OR multiple pauses in last 12 months |
| 9–10 | Withdrawal can be paused indefinitely OR exit requires curator action with no SLA |

#### X4 — Fee Structure

| Band | Pattern |
|---|---|
| 1–3 | Transparent fees: management ≤ 0.10%/year, performance ≤ 10%, no hidden costs |
| 4–5 | Standard: mgmt 0.5–1%, perf 15–20%, with documented hurdle rate |
| 6–7 | High fees (mgmt > 1% OR perf > 25%) OR fee structure undisclosed |
| 8–9 | Curator can change fees unilaterally without notice or governance approval |

#### X5 — Vault Maturity

| Band | Pattern |
|---|---|
| 1–2 | > 3 years live, > $500M peak TVL, zero loss-causing incidents |
| 3–4 | > 2 years live, > $100M TVL, no loss incidents |
| 5 | > 1 year live, > $10M TVL, clean track record |
| 6–7 | 6–12 months live OR $1M–$10M TVL |
| 8–9 | < 6 months live OR < $1M TVL OR recent loss-causing incident |
| 10 | < 1 month live or known critical vulnerability unpatched |

### 2.4 Hard caps (Type W specific)

In addition to base GRS > 7.5 hard cap:

- **X2 = 9** (plain EOA atomist on drain-vector) → likely Type W EXCLUDED if not mitigated by v3.9 Tier B attestation
- **X5 ≥ 9** combined with X2 ≥ 8 → EXCLUDED (immature wrapper + weak custody)
- **X3 = 10** (indefinite pause possible) → mandatory CEO review

### 2.5 GRS computation (unchanged formula, new SSR input)

```
SSR_W = 0.25·X1 + 0.30·X2 + 0.20·X3 + 0.15·X4 + 0.10·X5
GRS = 0.35·protocol_risk + 0.25·asset_risk + 0.40·SSR_W
```

`protocol_risk` continues via v4.0 Multi-Protocol Rule extension (chain + protocols).
`asset_risk` per existing Layer 2 ARS aggregation.

---

## 3. Migration of existing wrapper strategies

Four currently-active strategies in Hallmark should be reclassified from Type 1 / Type 3 to Type W:

| Strategy | Old Type | Old GRS | Atomist Pattern | Expected X2 |
|---|---|---|---|---|
| `fusnsteth-ipor-fusion-ethereum` | 1 | 4.96 | IPOR DAO 4-of-7 multisig (after v3.9 recheck) | 3–4 |
| `taucbeth-ipor-fusion-base` | 1 | 5.10 (cascade-excluded) | Single EOA `0xd556a9fa…` | 9 (EXCLUDED) |
| `ipsrbtc-ipor-fusion-ethereum` | 1 | 10.0 (double cascade-excluded via Reservoir L1) | Likely single EOA (per Reservoir pattern) | 9 + cascade |
| `gteusdc-morpho` | 3 | (TBD) | Gauntlet (institutional curator, multisig) | 4–5 |

Migration is **per-strategy re-scoring** under v4.1, not a mechanical bulk transform — each requires fresh evaluation of X1–X5 evidence.

---

## 4. Schema changes

### 4.1 `strategy.schema.json`

- Add `"W"` to the `strategy_type` enum
- Allow `^x[1-9]$` keys in `strategy_specific_criteria` (in addition to existing `^s[1-9]$`)

The two key namespaces (s1–s5 and x1–x5) are **mutually exclusive per strategy** — Types 1/2/3 use s-keys, Type W uses x-keys.

### 4.2 No new YAML files required

Type W strategies live in the same `public/scores/strategies/` directory as other types. The `strategy_type` field distinguishes them.

---

## 5. Frontend integration

- `STRATEGY_TYPE_DISPLAY`: add `"W": "Wrapper Vault"`
- `S_FACTOR_LABELS` becomes `STRATEGY_FACTOR_LABELS` (covers both s- and x-namespaces)
- `GRSPanel` reads `strategy.strategy_specific_criteria` and looks up labels via `STRATEGY_FACTOR_LABELS[strategy_type]`

---

## 6. Changelog entry (append to `risk_methodology.md` §7)

```
| 4.1 | 2026-05-18 | NEW STRATEGY TYPE — Type W (Wrapper Vault).
Addresses the methodology gap where vault-wrapped strategies (Ipor Fusion,
Morpho MetaMorpho, Yearn V3, generic 4626 wrappers) were forced into
Types 1/2/3 that didn't capture the two-layer trust model (depositor
delegates execution to curator/atomist). Introduces X1–X5 rubric:
X1 Underlying Strategy Risk, X2 Curator/Atomist Trust (30% weight —
the differentiator), X3 Exit Mechanism, X4 Fee Structure, X5 Vault
Maturity. X2 = 9 (plain EOA atomist) creates near-hard-cap consistent
with v3.9 Tier A treatment. Initial migration: fusnstETH, TAUCBETH,
ipsrBTC (Ipor Fusion), gteusdc-morpho (MetaMorpho curated). GRS formula
unchanged — SSR_W = weighted sum of X1–X5. Schema extended: "W" added
to strategy_type enum, x1-x5 keys allowed in strategy_specific_criteria.
See methodology/v41_amendment.md.
```
