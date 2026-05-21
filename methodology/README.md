# The Hallmark — Methodology

**Current version:** v4.1 (May 2026) · see [Amendments timeline](#amendments-timeline) for the full evolution.

The Hallmark is a four-layer risk methodology for evaluating DeFi yield strategies. Every chain, protocol, asset, and yield strategy in ForgeYields vaults receives a numerical score derived from this framework.

## Structure

```
Layer 1 — Protocol Risk Score (PRS)    → see layer1-protocol.md
Layer 2 — Asset Risk Score (ARS)       → see layer2-asset.md
Layer 3 — Global Risk Score (GRS)      → see layer3-strategy.md
Caps & Concentration Rules             → see caps.md
Formulas                               → see formulas.md
```

## Amendments timeline

| Version | Date | Title | Trigger |
|---------|------|-------|---------|
| v3.4 | 2026-04-08 | Base methodology (3-layer cascade, S-factors per type) | Foundation |
| v3.5 | 2026-04-22 | `max(C4a, C4 composite) ≥ 8` catastrophic-incident floor | Curve V1 Vyper exploit retrospective |
| v3.6 | 2026-04-30 | C5 cross-chain DVN + C1 audit-coverage gap rule | Kelp DAO 1-of-1 DVN exploit precedent |
| v3.7 | 2026-05-06 | A3 Async Exit Credit + WATCHLIST band 6.0–6.5 | sUSDS atomic-redeem + cUSD queue assessments |
| v3.8 | 2026-05-07 | L3 Recursive Strategy Collateral Rule | Pendle PT-sUSDe as Morpho collateral pattern |
| v3.9 | 2026-05-07 | L1 C3 Custody Mode Recognition (MPC threshold-signature) | Saturn Fireblocks MPC attestation precedent |
| v3.10 | 2026-05-08 | L2 §A4 RWA Backing Class Taxonomy (T / C / I) | sUSDat / Altura / USD.AI rescore observations |
| v4.0 | 2026-05-18 | Layer 0 Chain Risk Score (CRS, N1–N5 rubric) + integration via existing Multi-Protocol Rule | Monad / HyperEVM / Plasma chain-novelty under-rating |
| v4.1 | 2026-05-18 | L3 Type W (Wrapper Vault, X1–X5 rubric) for delegated-execution strategies | Ipor Fusion + MetaMorpho-style two-layer trust model |

Full amendment text: see `amendments/` subdirectory.

## Core principles

1. **Cascade gating.** A strategy can only score as well as its weakest underlying component. Layer 3 inherits exclusions from Layer 1 and Layer 2.

2. **Leading indicators over lagging ones.** The methodology prioritizes structural red flags (custody configuration, governance keys, audit gaps) that are observable *before* incidents, not just lagging indicators like exploit history.

3. **Non-compensable risk factors.** Certain risks — anonymous teams, single-key custody on drain-vector roles, sustained peg loss — are not offsettable by strong scores elsewhere. They trigger automatic exclusion regardless of weighted average.

## Score scale

All scores use a 1–10 scale where **1 = lowest risk** and **10 = highest risk**. The same scale applies to PRS, ARS, GRS, and the underlying C/A/S criteria.

## How scores are produced

The full assessment process for each score, including on-chain verification and rationale, is maintained in a private repository. Institutional limited partners may request access under a signed disclosure agreement.

This public repository contains the methodology framework and the resulting numerical scores. The numerical scores are sufficient to verify that the methodology has been applied; the deeper rationale is gated.
