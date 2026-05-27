# The Hallmark

**The Hallmark is the open-source risk methodology of [ForgeYields](https://forgeyields.com).**

We score every chain, protocol, asset, and yield strategy in our vaults against a transparent, four-layer framework. The methodology is published here in full. The numerical scores are too.

> **Current version:** v4.3 (May 2026). See the [Amendments](#amendments) section below for the full version timeline.

---

## What you'll find here

```
methodology/         The framework — rubrics, formulas, amendments
schemas/             JSON Schemas defining the score structure
scores/              Numerical scores per chain, protocol, asset, and strategy
```

This repository is auto-mirrored from our private working repo. Any change to a score or rubric is reflected here within minutes of the corresponding PR being merged.

---

## The four layers

### Layer 0 — Chain Risk Score (CRS) — *v4.0*

Every chain on which ForgeYields deploys strategies receives a CRS from 1 (lowest risk) to 10 (highest risk). Five N-criteria:

| Criterion | What it measures |
|-----------|------------------|
| N1 — Consensus & Decentralization | Validator/sequencer count, client diversity, finality mechanism |
| N2 — Sequencer Architecture | L1 vs L2 sequencer model, decentralization, escape hatches |
| N3 — Operational Track Record | Years live, sustained halts, recovery patterns |
| N4 — Bridge Architecture | Native canonical bridge security model |
| N5 — VM Maturity | Years in production, formal semantics, audit chain on upgrades |

Chain risk feeds Layer 1 Protocol Risk via the Multi-Protocol Rule rather than getting its own composite weight — so per-chain deployments of the same protocol (Morpho on Ethereum vs Morpho on Monad) score differently.

Full rubric: [`methodology/amendments/v40-amendment.md`](./methodology/amendments/v40-amendment.md)

### Layer 1 — Protocol Risk Score (PRS)

Every DeFi protocol that ForgeYields strategies depend on receives a PRS from 1 to 10. Six criteria:

| Criterion | Weight | What it measures |
|-----------|--------|------------------|
| C1 — Audit Status | 25% | Audit count, auditor tier, findings resolution, coverage gaps |
| C2 — TVL History | 10% | Absolute TVL, 30-day trend, distance from ATH |
| C3 — Governance Quality | 20% | Multisig configuration, timelock, admin key custody (v3.9 MPC recognition) |
| C4 — Incident History & Operational Age | 20% | Past exploits, loss magnitude, response quality, mainnet age |
| C5 — Smart Contract Risk | 15% | Code complexity, dependencies, upgradeability, cross-chain config (v3.6 DVN rules) |
| C6 — Team & Transparency | 10% | Doxxing status, public track record, communication |

Full rubric: [`methodology/layer1_protocol_assessment_methodology.md`](./methodology/layer1_protocol_assessment_methodology.md)

### Layer 2 — Asset Risk Score (ARS)

Every underlying asset (USDC, USDat, sUSDai, wstETH, etc.) receives an ARS from 1 to 10. Five criteria:

| Criterion | Weight | What it measures |
|-----------|--------|------------------|
| A1 — Peg Mechanism | 20% | How the asset's value is maintained |
| A2 — Depeg History | 25% | Observed deviation from target NAV |
| A3 — Liquidity Depth | 25% | On-chain DEX depth, async exit credit (v3.7) |
| A4 — Collateral Backing | 20% | Reserve quality, attestation, custody (v3.10 Class T/C/I taxonomy) |
| A5 — Market Cap / Supply Concentration | 10% | Mcap, top wallet concentration |

Full rubric: [`methodology/layer2_asset_assessment_methodology.md`](./methodology/layer2_asset_assessment_methodology.md)

### Layer 3 — Global Risk Score (GRS)

Each yield strategy receives a GRS from 1 to 10. The GRS composes the three preceding layers:

```
GRS = 0.35 × ProtocolRisk + 0.25 × AssetRisk + 0.40 × StrategySpecificRisk
```

Strategy-specific risk decomposes into S-factors (or X-factors for Type W) that depend on the strategy type:

- **Type 1** — Looping / Leverage (S1–S4)
- **Type 2A** — LP / AMM Classic (S1–S4)
- **Type 2B** — LP / PT Pendle (S1–S4)
- **Type 2C** — Pendle PT bond-like (S1–S4)
- **Type 3** — Lending vault (S1–S5)
- **Type W** — Wrapper Vault — *v4.1* (X1–X5, including X2 Curator/Atomist Trust at 30% weight)

Type W was introduced in v4.1 to capture the two-layer trust model of delegated-execution wrappers like Ipor Fusion and MetaMorpho-curated vaults, which Types 1/2/3 do not cleanly express.

Full rubric: [`methodology/layer3_strategy_assessment_methodology.md`](./methodology/layer3_strategy_assessment_methodology.md)
Type W amendment: [`methodology/amendments/v41-amendment.md`](./methodology/amendments/v41-amendment.md)

### Eligibility — composite + hard caps

A strategy is **EXCLUDED** if either condition trips:

1. **Composite cutoff:** GRS > 7.5
2. **Hard caps:** any single criterion threshold crossing — e.g., A2 ≥ 9 (sustained depeg), C3 ≥ 8 (single-EOA on drain-vector role), X2 = 9 (plain-EOA atomist on a wrapper), C5 ≥ 9 (1-of-1 DVN cross-chain)

A clean composite GRS is not sufficient. Hard caps override.

---

## How to read a score

Each `.yaml` file in `scores/` follows the JSON Schema defined in `schemas/`. Scores are numerical only — interpretation (whether a score qualifies for deployment, what cap applies, etc.) is governed by the published methodology rubrics in `methodology/` and applied internally by ForgeYields.

Example — `scores/strategies/gteusdc-morpho.yaml`:

```yaml
name: gteUSDc Morpho (Gauntlet eUSD Core)
slug: gteusdc-morpho
chain: ethereum
strategy_type: W
vault_target: fyUSDC
assessment:
  date: 2026-05-18
  methodology_version: v4.1
grs: 3.63
components:
  protocol_risk: 2.2
  asset_risk: 4.4
  strategy_specific_risk: 4.40
strategy_specific_criteria:
  x1: 4
  x2: 5
  x3: 4
  x4: 4
  x5: 5
dependencies:
  chains: [ethereum]
  protocols: [morpho-blue]
  assets: [eusd, wbtc, eth-plus, wsteth]
```

GRS verification: `0.35 × 2.2 + 0.25 × 4.4 + 0.40 × 4.40 = 3.63`. The methodology documents in `methodology/` (and amendments) explain how each criterion is evaluated, including how chain risk feeds PR via the Multi-Protocol Rule and how X-criteria are weighted for Type W.

---

## Amendments

The methodology evolves as DeFi evolves. Each amendment is published with its rationale and trigger event.

| Version | Date | Title |
|---------|------|-------|
| v3.7 | 2026-05-06 | A3 Async Exit Credit + WATCHLIST band 6.0–6.5 |
| v3.8 | 2026-05-07 | L3 Recursive Strategy Collateral Rule |
| v3.9 | 2026-05-07 | L1 C3 Custody Mode Recognition (MPC threshold-signature) |
| v3.10 | 2026-05-08 | L2 §A4 RWA Backing Class Taxonomy (T / C / I) |
| **v4.0** | **2026-05-18** | **Layer 0 Chain Risk Score (CRS, N1–N5 rubric) + integration via Multi-Protocol Rule** |
| **v4.1** | **2026-05-18** | **L3 Type W (Wrapper Vault, X1–X5 rubric) for delegated-execution strategies** |
| **v4.2** | **2026-05-22** | **L3 procedural clarification: GRS always computed; verdict derived separately (cascade exclusion ≠ skip computation)** |
| **v4.3** | **2026-05-22** | **L1 C3 Tier B+ "Regulated Public Custody" carve-out (narrow exception to v3.9 single-EOA hard cap for SEC-listed issuers with SOC 1/2 + banking regulator)** |

See [`methodology/amendments/`](./methodology/amendments/) for full text.

---

## License

Methodology framework and numerical scores in this repository are licensed under **CC-BY-4.0**. You may use, share, and adapt with attribution.

---

## More

- **ForgeYields:** [forgeyields.com](https://forgeyields.com)
- **Live docs (rendered):** [forge-labs.gitbook.io/forge-docs/hallmark/overview](https://forge-labs.gitbook.io/forge-docs/hallmark/overview)
- **Audited assessments:** ForgeYields maintains a private repository of full deep-dive assessments behind each numerical score. Institutional LPs and integrators may request read-only access. Contact: forge.fi.contact@gmail.com
- **Issues / PRs:** This repo is auto-mirrored and read-only. Issues or methodology suggestions can be submitted via the ForgeYields contact channel.
