# The Hallmark

**The Hallmark is the open-source risk methodology of [ForgeYields](https://forgeyields.com).**

We score every protocol, asset, and yield strategy in our vaults against a transparent, three-layer framework. The methodology is published here in full. The numerical scores are too.

---

## What you'll find here

```
methodology/         The framework — rubrics, formulas, amendments
schemas/             JSON Schemas defining the score structure
scores/              Numerical scores per protocol, asset, and strategy
```

This repository is auto-mirrored from our private working repo. Any change to a score or rubric is reflected here within minutes of the corresponding PR being merged.

---

## The three layers

### Layer 1 — Protocol Risk Score (PRS)

Every DeFi protocol that ForgeYields strategies depend on receives a PRS from 1 (lowest risk) to 10 (highest risk). The PRS is the weighted average of six criteria:

| Criterion | Weight | What it measures |
|-----------|--------|------------------|
| C1 — Audit Status | 25% | Audit count, auditor tier, findings resolution, coverage gaps |
| C2 — TVL History | 10% | Absolute TVL, 30-day trend, distance from ATH |
| C3 — Governance Quality | 20% | Multisig configuration, timelock, admin key custody |
| C4 — Incident History & Operational Age | 20% | Past exploits, loss magnitude, response quality, mainnet age |
| C5 — Smart Contract Risk | 15% | Code complexity, dependencies, upgradeability, cross-chain config |
| C6 — Team & Transparency | 10% | Doxxing status, public track record, communication |

Full rubric: [`methodology/v3.10/layer1-protocol.md`](./methodology/v3.10/layer1-protocol.md)

### Layer 2 — Asset Risk Score (ARS)

Every underlying asset (USDC, USDat, sUSDai, wstETH, etc.) receives an ARS from 1 to 10. Five criteria:

| Criterion | Weight | What it measures |
|-----------|--------|------------------|
| A1 — Peg Mechanism | 20% | How the asset's value is maintained |
| A2 — Depeg History | 25% | Observed deviation from target NAV |
| A3 — Liquidity Depth | 25% | On-chain DEX depth, async exit credit |
| A4 — Collateral Backing | 20% | Reserve quality, attestation, custody (v3.10 Class T/C/I taxonomy) |
| A5 — Market Cap / Supply Concentration | 10% | Mcap, top wallet concentration |

Full rubric: [`methodology/v3.10/layer2-asset.md`](./methodology/v3.10/layer2-asset.md)

### Layer 3 — Global Risk Score (GRS)

Each yield strategy receives a GRS from 1 to 10. The GRS is the weighted composition of three components:

```
GRS = 0.35 × ProtocolRisk + 0.25 × AssetRisk + 0.40 × StrategySpecificRisk
```

Strategy-specific risk decomposes into S-factors that depend on the strategy type:
- **Type 1** — Looping / Leverage
- **Type 2A** — LP / AMM Classic
- **Type 2B** — LP / PT Pendle
- **Type 2C** — Pendle PT bond-like
- **Type 3** — Lending vault

Full rubric: [`methodology/v3.10/layer3-strategy.md`](./methodology/v3.10/layer3-strategy.md)

---

## How to read a score

Each `.yaml` file in `scores/` follows the JSON Schema defined in `schemas/`. Scores are numerical only — interpretation (whether a score qualifies for deployment, what cap applies, etc.) is governed by the published methodology rubrics in `methodology/` and applied internally by ForgeYields.

Example — `scores/protocols/saturn.yaml`:

```yaml
name: Saturn Protocol
slug: saturn
chains: [ethereum]
assessment:
  date: 2026-05-08
  methodology_version: v3.10
prs: 5.47
criteria:
  c1: 5
  c2: 8
  c3: 7
  c4: 4.10
  c5: 6
  c6: 3
```

The score `5.47` is the result of `0.25·5 + 0.10·8 + 0.20·7 + 0.20·4.10 + 0.15·6 + 0.10·3`. The methodology documents in `methodology/v3.10/` explain how each criterion is evaluated.

---

## Amendments

The methodology evolves as DeFi evolves. Each amendment is published with its rationale and trigger event.

| Version | Date | Title |
|---------|------|-------|
| v3.7 | 2026-05-06 | A3 Async Exit Credit + WATCHLIST band 6.0–6.5 |
| v3.8 | 2026-05-07 | L3 Recursive Strategy Collateral Rule |
| v3.9 | 2026-05-07 | L1 C3 Custody Mode Recognition (MPC threshold-signature) |
| v3.10 | 2026-05-08 | L2 §A4 RWA Backing Class Taxonomy (T / C / I) |

See [`methodology/v3.10/amendments/`](./methodology/v3.10/amendments/) for full text.

---

## License

Methodology framework and numerical scores in this repository are licensed under **CC-BY-4.0**. You may use, share, and adapt with attribution.

---

## More

- **ForgeYields:** [forgeyields.com](https://forgeyields.com)
- **Audited assessments:** ForgeYields maintains a private repository of full deep-dive assessments behind each numerical score. Institutional LPs may request read-only access under a signed disclosure agreement. Contact: forge.fi.contact@gmail.com
- **Issues / PRs:** This repo is auto-mirrored and read-only. Issues or methodology suggestions can be submitted via the ForgeYields contact channel.
