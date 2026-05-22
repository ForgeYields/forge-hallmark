# ForgeYields — DeFi Risk Assessment Methodology

**Current version:** v4.1 (May 2026)
**Last consolidated into this single-file snapshot:** 2026-05-07 at v3.9 baseline.
**Active layered amendments:** v3.10, v4.0, v4.1 — see `amendments/` for full text; these have not yet been folded into this consolidated framework file but are operationally in force.
**Author:** Risk Analyst Agent (updated by CEO)
**Audience:** Institutional LPs, Quant Funds, Internal Strategy Team
**Scope:** All yield strategies under consideration for fyUSDC, fyETH, fyWBTC vaults
**Amendment history:** v3.5 (C4 max rule) + v3.6 (C5 cross-chain + C1 audit coverage) + v3.7 (L2 A3 async exit credit + WATCHLIST band 6.0–6.5) + v3.8 (L3 Recursive Strategy Collateral Rule) + v3.9 (L1 C3 Custody Mode Recognition: MPC threshold-signature) + v3.10 (L2 §A4 RWA Backing Class Taxonomy T/C/I) + **v4.0 (Layer 0 Chain Risk Score)** + **v4.1 (L3 Type W Wrapper Vault, X1–X5 rubric)** — see §7 Changelog and `amendments/`.

---

## 1. Executive Summary

This document defines the quantitative risk framework used by ForgeYields to evaluate, score, and monitor every yield strategy deployed across our vaults. The framework operates in three layers:

| Layer | Scope | Output |
|-------|-------|--------|
| **Layer 1 — Protocol Registry** | Per-protocol assessment | Protocol Risk Score (1–10) |
| **Layer 2 — Asset Registry** | Per-asset assessment | Asset Risk Score (1–10) |
| **Layer 3 — Strategy Assessment** | Per-strategy composite | Global Strategy Risk Score (1–10) |

**Scoring convention:** 1 = lowest risk (highest confidence), 10 = highest risk (lowest confidence). A strategy must score ≤ 7.5 to be eligible for vault deployment, subject to Section 5 concentration and position sizing rules. Strategies scoring > 7.5 are excluded.

---

## 2. Layer 1 — Protocol Registry

### 2.1 Purpose

Assess the systemic risk of each DeFi protocol that ForgeYields interacts with or considers integrating. Each protocol is scored once and re-evaluated quarterly or upon material events (exploit, governance change, major upgrade).

Bridge risk is captured within Protocol Risk (C5 Smart Contract Risk) and Asset Risk (A4 Collateral Backing). Canonical bridges (ETH L1↔L2) score higher than third-party bridges. CCTP (Circle) and LayerZero DVN are assessed via their underlying validator sets and attestation mechanisms.

### 2.2 Criteria and Weightings

| # | Criterion | Weight | Score Range | Description |
|---|-----------|--------|-------------|-------------|
| C1 | **Audit Status** | 25% | 1–10 | Number of audits, auditor reputation, recency, scope coverage |
| C2 | **TVL History** | 10% | 1–10 | Current TVL, 30-day trend, distance from ATH, TVL stability |
| C3 | **Governance Quality** | 20% | 1–10 | Governance mechanism, timelock duration, upgrade controls |
| C4 | **Incident History & Operational Age** | 20% | 1–10 | Past exploits, loss magnitude, response time, remediation + operational track record |
| C5 | **Smart Contract Risk** | 15% | 1–10 | Code complexity, external dependencies, upgradeability pattern |
| C6 | **Team & Transparency** | 10% | 1–10 | Team doxxing status, public reputation, communication consistency |

**Protocol Risk Score** = 0.25·C1 + 0.10·C2 + 0.20·C3 + 0.20·C4 + 0.15·C5 + 0.10·C6

### 2.3 Scoring Rubrics

> **Rubric vs. operational proxy distinction:** Rubrics in this section define *what* each score band means qualitatively. Layer-specific methodology files define *how* to operationalize each rubric (proxy metrics, data sources, verification steps). Operational proxies (e.g., LOC thresholds for C5) are implementation details — they do not modify the rubric definitions here. In case of conflict, rubric definitions in this document take precedence.

#### C1 — Audit Status (Weight: 25%)

**Pre-scoring step — Upgradeability classification (required)**

Before applying any scoring rubric, classify the protocol's core contracts:

- **Type A (immutable core):** Core logic is non-upgradeable (no proxy, no admin upgrade key). Audit age is NOT penalized. Score on audit quality, auditor tier, findings resolution, and Unaudited Code Delta (UCD).
- **Type B (upgradeable core):** Core contracts use a proxy pattern or admin-controlled upgrade mechanism. Full rubric with audit recency applies.
- **Mixed:** Apply Type A to immutable components, Type B to upgradeable components. Use the more conservative (higher) score.

**Unaudited Code Delta (UCD):** Estimated proportion of deployed code (by functional scope) added or modified since the most recent audit. Includes new integrations, wrappers, reward routing, and any post-audit additions. Expressed as a percentage.

---

**TYPE A — IMMUTABLE CORE (audit recency not penalized):**

| Score | Criteria |
|-------|----------|
| 1–2 | ≥ 3 audits from top-tier firms, all critical/high findings resolved, formal verification present, UCD = 0% |
| 3–4 | 2+ audits from reputable firms, no unresolved critical/high findings, UCD < 20% |
| 5–6 | 1 reputable audit, no unresolved critical/high; OR medium findings only; OR UCD 20–40% |
| 7–8 | 1 audit from a lesser-known firm; OR unresolved high finding; OR UCD > 40% |
| 9–10 | No audit; OR unrecognized firm; OR unresolved critical finding; OR UCD > 60% |

**TYPE B — UPGRADEABLE CORE (full recency rubric):**

| Score | Criteria |
|-------|----------|
| 1–2 | ≥ 3 audits from top-tier firms (Trail of Bits, OpenZeppelin, Spearbit, Cantina), most recent < 6 months ago, all critical findings resolved, formal verification present |
| 3–4 | 2 audits from reputable firms, most recent < 12 months ago, no unresolved critical/high findings |
| 5–6 | 1 audit from a reputable firm, most recent < 18 months ago; or 2+ audits with some unresolved medium findings |
| 7–8 | 1 audit from a lesser-known firm; or most recent audit > 18 months ago; or unresolved high findings |
| 9–10 | No audit; or audit from an unrecognized firm; or critical unresolved findings |

#### C2 — TVL History (Weight: 10%)

> **v3.4 update (FORA-141):** The prior rubric used ATH distance as a co-equal scoring axis, which disproportionately penalised large protocols during macro drawdowns (a $6.59B protocol scored the same as a $50M one if both were equidistant from ATH). The revised model makes absolute TVL the primary axis with ATH distance and 30d drawdown as secondary modifiers only.

**STEP 1 — Base score from absolute TVL (primary axis):**

| TVL           | Base Score |
|---------------|------------|
| > $1B         | 1          |
| $500M – $1B   | 2          |
| $250M – $500M | 4          |
| $100M – $250M | 6          |
| $10M – $100M  | 8          |
| < $10M        | 10         |

**STEP 2 — Add modifiers (secondary axes):**

| Condition                               | Modifier |
|-----------------------------------------|----------|
| 30d drawdown > 20% (TVL fell > 20%)     | +1       |
| 30d drawdown > 40% (TVL fell > 40%)     | +2       |
| ATH distance < 20% (current/ATH < 20%) | +1       |
| ATH distance < 10% (current/ATH < 10%) | +2       |

**Notes:**
- The 30d drawdown conditions are mutually exclusive (apply the highest applicable modifier, not both).
- The ATH distance conditions are mutually exclusive (apply the highest applicable modifier, not both).
- The two modifier groups (drawdown and ATH distance) stack independently. Maximum combined modifier = **+4** (drawdown > 40% AND ATH distance < 10%).

**STEP 3 — Final C2 score = min(base + modifiers, 10)**

**Worked examples:**
- Ethena ($6.59B TVL, current/ATH 44%, 30d trend −1%): base 1 + ATH ≥20% (+0) + trend ≤20% (+0) = **C2 = 1**
- Convex ($626M TVL, current/ATH 3%, 30d −9.6%): base 2 + ATH <10% (+2) + trend ≤20% (+0) = **C2 = 4**
- $50M protocol at 3% of ATH, −5% trend: base 8 + ATH <10% (+2) + trend ≤20% (+0) = **C2 = 10**

#### C3 — Governance Quality (Weight: 20%)

| Score | Criteria |
|-------|----------|
| 1–2 | On-chain DAO governance, timelock ≥ 48h, no admin keys or admin behind 4/7+ multisig with known signers, immutable core contracts |
| 3–4 | On-chain governance with timelock ≥ 24h, admin multisig 3/5+ with known signers, upgradeable via proxy with timelock |
| 5–6 | Multisig governance (3/5+) with timelock ≥ 12h, partially upgradeable contracts |
| 7–8 | Multisig governance (2/3 or fewer signers), timelock < 12h, or anonymous signers |
| 9–10 | Single admin key, no timelock, or fully anonymous governance, or unverified multisig |

Note on operational security: C3 scores governance structure (multisig configuration, timelock duration). Operational security practices (HSM usage, key rotation policies, access controls) are assessed qualitatively during due diligence but are not quantitatively scored in C3. Protocols with known weak key management practices (cloud storage without HSM, no key rotation policy, excessive access privileges) should be escalated for manual review and may be excluded regardless of C3 score.

#### C4 — Incident History & Operational Age (Weight: 20%)

C4 is a composite score: **C4 = 0.70 · C4a (Incident History) + 0.30 · C4b (Operational Age)**

Scoring process: Score C4a and C4b independently using the rubrics below, then combine:
C4 = 0.70 × C4a + 0.30 × C4b

**C4a — Incident History (70% of C4):**

| Score | Criteria |
|-------|----------|
| 1–2 | Zero exploits, active bug bounty ≥ $1M |
| 3–4 | Zero exploits, or minor incident (< $100K loss) with full recovery and < 24h response |
| 5–6 | One moderate incident ($100K–$5M loss) with full user compensation, post-mortem published, remediation verified |
| 7–8 | One major incident ($5M–$50M loss) or multiple moderate incidents, partial user compensation |
| 9–10 | Major exploit (> $50M loss), incomplete compensation, repeated incidents, or no post-mortem published |

**C4b — Operational Age (30% of C4):**

| Score | Criteria |
|-------|----------|
| 1–2 | > 3 years operational history on mainnet |
| 3–4 | 2–3 years operational history |
| 5–6 | 1–2 years operational history |
| 7–8 | 6–12 months operational history |
| 9–10 | < 6 months operational history |

#### C5 — Smart Contract Risk (Weight: 15%)

| Score | Criteria |
|-------|----------|
| 1–2 | Simple, battle-tested architecture (e.g., Compound V2 fork), minimal external dependencies, non-upgradeable or transparent proxy with timelock, verified source on Etherscan |
| 3–4 | Moderate complexity, well-documented, < 5 external dependencies, upgradeable with timelock, all dependencies audited |
| 5–6 | Moderate-to-high complexity, 5–10 external dependencies, some unaudited dependencies, novel but tested patterns |
| 7–8 | High complexity, > 10 external dependencies, novel unproven patterns, or significant reliance on off-chain components. Protocols with critical dependencies on off-chain infrastructure (CEX accounts for delta hedging, cloud services for core operations) score minimum 7, as such dependencies introduce counterparty and operational risks not mitigated by smart contract audits. |
| 9–10 | Extremely complex, unverified source code, heavy reliance on external oracles without fallbacks, or experimental architecture |

#### C6 — Team & Transparency (Weight: 10%)

| Score | Criteria |
|-------|----------|
| 1–2 | Fully doxxed team with reputable background (prior successful projects, known industry track record) |
| 3–4 | Partially doxxed team with verifiable track record in DeFi/crypto |
| 5–6 | Consistent pseudonymous presence with established history and regular public communication |
| 7–8 | Anonymous team with irregular communication, limited public engagement |
| 9–10 | Fully anonymous, no public history, no verifiable track record, no consistent communication |

### 2.4 Data Sources

- **Audits:** Protocol documentation, auditor public reports, Solodit, Code4rena archives
- **TVL:** DefiLlama (primary), DeFi Pulse (cross-reference)
- **Governance:** On-chain analysis (Etherscan/Blockscout), Tally, Snapshot, protocol documentation
- **Incidents:** Rekt.news, DeFi incident databases, protocol post-mortems, Chainalysis reports
- **Smart Contracts:** Etherscan verified source, GitHub repositories, dependency analysis tools
- **Team:** Protocol websites, LinkedIn, Twitter/X, conference appearances, prior project history

In case of >20% divergence between data sources, manual verification is required. The most conservative estimate is selected.

---

## 3. Layer 2 — Asset Registry

### 3.0 Asset Classification Rules

For yield-bearing stablecoins and complex structured assets, apply the following boundary rules to determine which risk layer captures each dimension:

| Risk Dimension | Layer | Criterion |
|---|---|---|
| Smart contract risk of the issuer | Layer 1 | C5 |
| Governance risk of the issuer | Layer 1 | C3 |
| Incident history of the issuer | Layer 1 | C4 |
| Peg stability of the token | Layer 2 | A2 |
| Market liquidity of the token | Layer 2 | A3 |

**Collateral backing (A4) — conditional treatment:**

- If backing is held in **static reserves** (cash, T-bills, ETH) → score **Layer 2, A4 only**.
- If backing mechanism **IS the protocol's active operation** (e.g., Ethena delta-neutral positions, RWA invoice management) → score A4 **AND** flag a cross-dependency with Layer 1 C5/C4. **Do not double-penalize** — note the dependency explicitly in the assessment and apply the more conservative of the two resulting scores.

**Dual-assessment requirement:** The following asset types always require a Layer 1 assessment of their issuing protocol in addition to the Layer 2 asset assessment:

- Yield-bearing stablecoins (USR, sUSDe, pmUSD, etc.)
- RWA-backed tokens
- Protocol-native tokens used as collateral

When both assessments are required, the final Layer 2 Asset Risk Score is computed normally. The Layer 1 Protocol Risk Score for the issuer is then carried forward into any Layer 3 Strategy Assessment that uses the asset, ensuring both dimensions are reflected in the composite strategy risk score.

---

### 3.1 Purpose

Assess the risk profile of each underlying asset used within yield strategies. Assets are scored independently and re-evaluated monthly or upon material events (depeg, reserve attestation failure, liquidity shock).

**Note on yield-bearing stablecoins:** For yield-bearing stablecoins (USR, sUSDe, pmUSD, etc.), the issuing protocol is assessed separately in Layer 1. Layer 2 scores only the asset itself — peg stability, liquidity, and backing. Example: for Resolv's USR, admin key minting risk is evaluated in Layer 1 (Protocol Registry), while USR peg stability and liquidity are evaluated here in Layer 2.

### 3.2 Criteria and Weightings

| # | Criterion | Weight | Score Range | Description |
|---|-----------|--------|-------------|-------------|
| A1 | **Peg Mechanism** | 20% | 1–10 | Type, robustness, and transparency of the peg/value mechanism |
| A2 | **Depeg History** | 25% | 1–10 | Historical max deviation, duration, frequency of depeg events |
| A3 | **Liquidity Depth** | 25% | 1–10 | DEX + CEX depth, slippage at institutional sizes ($1M+) |
| A4 | **Collateral Backing** | 20% | 1–10 | Reserve composition, attestation frequency, transparency |
| A5 | **Market Cap / Supply Concentration** | 10% | 1–10 | Market cap size and wallet concentration risk |

**Asset Risk Score** = 0.20·A1 + 0.25·A2 + 0.25·A3 + 0.20·A4 + 0.10·A5

### 3.3 Scoring Rubrics

#### A1 — Peg Mechanism (Weight: 20%)

| Score | Criteria |
|-------|----------|
| 1–2 | Native asset (ETH) or fully collateral-backed with 1:1 reserves in regulated custody (e.g., USDC by Circle), transparent attestation |
| 3–4 | Over-collateralized with liquid, diversified collateral (e.g., wstETH backed by staked ETH with Lido), redemption mechanism well-tested |
| 5–6 | Over-collateralized but concentration risk in collateral, or collateral includes volatile assets, or redemption delay > 7 days |
| 7–8 | Partially algorithmic with collateral backstop, or custodial with infrequent attestation (< monthly), or novel peg mechanism < 1 year old |
| 9–10 | Fully algorithmic peg, or unattested reserves, or opaque custodial arrangement, or no clear redemption mechanism |

#### A2 — Depeg History (Weight: 25%)

| Score | Criteria |
|-------|----------|
| 1–2 | Never depegged > 0.5% from target, or native asset with no peg dependency |
| 3–4 | Max deviation < 2%, duration < 4 hours, frequency ≤ 1 event in 2 years |
| 5–6 | Max deviation 2–5%, duration < 24 hours, frequency ≤ 2 events in 2 years |
| 7–8 | Max deviation 5–15%, or duration 1–7 days, or frequency > 2 events per year |
| 9–10 | Max deviation > 15%, or duration > 7 days, or sustained loss of peg |

#### A3 — Liquidity Depth (Weight: 25%)

| Score | Criteria |
|-------|----------|
| 1–2 | Can execute $10M+ swap with < 0.1% slippage, available on 5+ major DEXs/CEXs, 24h volume > $500M |
| 3–4 | Can execute $5M swap with < 0.25% slippage, available on 3+ major venues, 24h volume > $100M |
| 5–6 | Can execute $1M swap with < 0.5% slippage, available on 2+ major venues, 24h volume > $20M |
| 7–8 | $1M swap incurs 0.5–2% slippage, limited venue availability, 24h volume $1M–$20M |
| 9–10 | $1M swap incurs > 2% slippage, single venue dependency, 24h volume < $1M |

**Async Exit Credit (v3.7):** Assets with documented async withdrawal mechanisms (queue, cooldown, atomic redeem) receive an A3 modifier:

| Mechanism | SLA | A3 credit |
|-----------|-----|-----------|
| Atomic redeem (no wait) | SLA = 0 | −2 bands |
| Queue / cooldown | SLA ≤ 7 days | −1 band |
| Queue / cooldown | SLA ≤ 14 days | −1 band conditional on no pause history |
| Queue / cooldown | SLA > 14 days OR pause history within 12 months | no credit |

**Eligibility requirements for credit:**
- Documented in protocol docs (queue/cooldown/redeem mechanism with disclosed SLA)
- On-chain verifiable contract (queue contract address, redemption function selector)
- No history of withdrawal pause exceeding stated SLA in prior 12 months

**Caps on credit application:**
- A3 cannot be reduced below band 3 by async credit alone (preserves DEX/CEX failure-mode signal)
- Maximum total credit: −2 bands
- Credit nullified retroactively if a pause event occurs (asset re-scored on next monthly cadence)

**Rationale (v3.7):** A3 historically measured DEX/CEX market microstructure exclusively. Empirically, this created an asymmetric application: assets with deep DEX got implicit credit (queue mechanism redundant), while DEX-thin assets with strong queues were over-penalized. The async credit formalizes the redemption-path-as-exit-liquidity logic already implicit in atomic-redeem assets (e.g., sUSDS PSM 1:1) and aligns with peer methodologies (LlamaRisk, Gauntlet). Tail-risk (queue + DEX both fail simultaneously in stress) is preserved via the floor-band-3 cap and pause-history nullification rule. See changelog v3.7 entry for back-test.

#### A4 — Collateral Backing (Weight: 20%)

| Score | Criteria |
|-------|----------|
| 1–2 | 100%+ reserves in liquid, audited assets, real-time proof-of-reserves, monthly third-party attestation (Big 4 or equivalent), regulated custodian |
| 3–4 | 100%+ reserves, quarterly attestation by reputable firm, disclosed reserve composition, regulated custodian |
| 5–6 | Reserves ≥ 100% but attestation semi-annual or composition partially opaque, or reserves include illiquid assets (< 20%) |
| 7–8 | Reserves < 100% (under-collateralized), or attestation > 6 months old, or significant illiquid reserve component (20–50%) |
| 9–10 | No reserve attestation, or reserves significantly < 100%, or > 50% illiquid/opaque reserves, or no independent verification |

**RWA-backed asset override for A4:** For real-world asset (RWA) backed assets, replace the standard A4 rubric with the following criteria:

| Score | Criteria |
|-------|----------|
| 1–2 | Legally enforceable claim in multiple jurisdictions, liquidation timeline < 30 days, daily third-party valuation (mark-to-market), custodian in regulated jurisdiction (US, EU, CH) |
| 3–4 | Enforceable claim in one jurisdiction, liquidation 30–60 days, weekly independent valuation, custodian in regulated jurisdiction |
| 5–6 | Enforceable claim but untested in court, liquidation 60–90 days, monthly valuation, partially regulated custodian |
| 7–8 | Unclear legal enforceability, liquidation > 90 days, infrequent or self-reported valuation, offshore custodian |
| 9–10 | No clear legal claim, illiquid underlying with no defined liquidation path, no independent valuation, unregulated custodian |

#### A5 — Market Cap / Supply Concentration (Weight: 10%)

| Score | Criteria |
|-------|----------|
| 1–2 | Market cap > $10B, no single wallet holds > 5% of supply |
| 3–4 | Market cap $1B–$10B, top wallet holds < 10% of supply |
| 5–6 | Market cap $100M–$1B, some supply concentration but manageable |
| 7–8 | Market cap < $100M, or top wallet holds > 20% of supply |
| 9–10 | Highly concentrated supply, minimal market cap data, or no reliable supply data available |

**Note for native assets:** ETH scores A1=1, A2=1, A4=1, A5=1 by definition (no peg, no depeg, no collateral needed, massive market cap with distributed supply). A3 is scored normally based on market liquidity.

### 3.4 Data Sources

- **Peg Mechanism:** Protocol documentation, whitepapers, on-chain analysis
- **Depeg History:** CoinGecko/CoinMarketCap historical data, DeFi dashboards (Dune Analytics), custom monitoring
- **Liquidity:** 1inch/Paraswap aggregator quotes (DEX), order book snapshots from major CEXs, CoinGecko volume data
- **Collateral:** Proof-of-reserve dashboards, attestation reports, Nansen/Arkham on-chain analytics
- **Market Cap / Concentration:** CoinGecko, Etherscan token holder analysis, Nansen wallet concentration metrics

In case of >20% divergence between data sources, manual verification is required. The most conservative estimate is selected.

---

## 4. Layer 3 — Strategy Assessment

### 4.1 Purpose

Produce a composite risk score for each specific yield strategy by combining Layer 1 (protocol risk) and Layer 2 (asset risk) with strategy-type-specific risk factors. This is the final scoring layer that determines vault eligibility.

### 4.2 Composite Score Formula

**Global Risk Score (GRS)** = 0.35 × ProtocolRisk + 0.25 × AssetRisk + 0.40 × StrategySpecificRisk

#### 4.2.1 Weakest Link Rule

A protocol is **EXCLUDED** from deployment if **ANY** of the following conditions are met:

1. Protocol Risk Score (PRS) > 7.0
2. C1 (Audit Status) ≥ 9
3. C3 (Governance Quality) ≥ 8
4. **max(C4a, C4 composite) ≥ 8** (v3.5 amendment — incident history alone OR composite triggers)
5. C5 (Smart Contract Risk) ≥ 9 — **v3.6 additions:** 1-of-1 DVN / single-verifier cross-chain bridge → C5 ≥ 9 (hard cap); bridge threshold < 2-of-N or outside governance perimeter → C5 ≥ 8 (watchlist)
6. C6 (Team & Transparency) ≥ 8

**v3.6 C1 additions (audit coverage gap):**
- UCD > 50% on critical deployed surface → C1 ≥ 7
- Major production layer (bridge adapter, vault, oracle) entirely unaudited → C1 ≥ 8
- Acknowledged CRITICAL/HIGH finding on production contract > 90 days unfixed → C1 ≥ 7

An asset is **EXCLUDED** from deployment if **ANY** of the following conditions are met:

1. Asset Risk Score (ARS) > 6.5  *(v3.7: raised from 6.0; band 6.0–6.5 now WATCHLIST per Rule 3)*
2. A2 (Depeg History) ≥ 9

An asset with ARS in the band **6.0–6.5** is classified as **WATCHLIST** — eligible for deployment but capped at **15% concentration per vault** (per Section 5 Rule 3). Watchlist assets require monthly re-scoring (vs. the standard quarterly cadence) and a documented monitoring trigger list at the L2 assessment level.

**Rationale — non-compensable risk factors:**

**Human counterparty risk (C3, C4, C6):** Poor governance, major unresolved incidents, or anonymous unverified teams pose counterparty risk incompatible with trustless execution, regardless of technical quality.

**Fundamental technical risk (C1, C5):** Protocols without audits or with critical unresolved findings (C1 ≥ 9), or with unverified or experimental codebases (C5 ≥ 9), present execution risk that cannot be offset by strong governance or team reputation. Smart contract vulnerabilities are non-negotiable: no amount of process excellence compensates for exploitable code.

**Threshold justification:** The ≥ 8 threshold for C3, C4, C6 intentionally excludes weak multisig configurations (2/3 or fewer signers) and sub-12h timelocks. These governance structures provide insufficient security margin for protocols managing institutional capital. The ≥ 9 threshold for C1 and C5 targets only the most severe technical failures (no audit, unverified code) — moderate complexity or aging audits remain compensable.

**Rationale for asset hard cap:** An asset that has sustained a loss of peg for > 7 days (A2 ≥ 9) has demonstrated fundamental mechanism failure. Unlike temporary liquidity issues or short-lived depegs, prolonged peg loss indicates systemic backing problems that cannot be compensated by other factors.

**Rationale for v3.5 C4 amendment (`max(C4a, C4)`):** Under the v3.4 composite formula (`C4 = 0.70·C4a + 0.30·C4b`), a catastrophic incident (C4a=9) could be diluted below the hard cap threshold by long operational age. For example, a $292M exploit on a 2-year protocol yielded C4 composite = 7.50, failing to trigger the C4 hard cap despite the C4a=9 catastrophic rating. Per the non-compensable risk principle above, major incidents should NOT be compensable by age. v3.5 applies the C4 hard cap to `max(C4a, C4)` so that catastrophic C4a ≥ 8 triggers exclusion regardless of composite. See `methodology/v35_amendment.md` for full documentation.

**Rationale for v3.6 C5 cross-chain + C1 audit-coverage amendment:** C4 is a LAGGING indicator — it catches incidents after they happen. v3.6 adds LEADING indicators on the architectural preconditions that produce incidents. The Kelp DAO $292M Lazarus exploit (2026-04-18) is the canonical case: 1-of-1 LayerZero DVN was publicly documented + flagged 12 days before exploit, yet v3.4 methodology only scored C5=7 (off-chain floor). v3.6 forces C5 ≥ 9 on 1-of-1 DVN configurations, C5 ≥ 8 on bridge thresholds below 2-of-N, and C1 ≥ 7-8 on audit coverage gaps. Positive examples validating v3.6 is not over-broad: M^0 Foundation (Wormhole 13-of-19 Guardians) and Treehouse (Chainlink CCIP dual-committee) both pass cleanly. See `methodology/v36_amendment.md` for full documentation and back-test.

If re-scoring degrades an already-deployed strategy to trigger any exclusion condition, initiate exit within 7 days.

*Note: This is consistent with Section 5 Rules 2 and 3 which set 0% allocation for protocols scoring > 7.0 and assets scoring > 6.0.*

#### 4.2.2 Multi-Protocol Rule

When a strategy involves multiple protocols:

**ProtocolRisk = max(Pi) + (N-1) × 0.20 × mean(Pi)**

Where N = number of protocols involved and Pi = individual protocol risk scores.

**Example** (Curve + Convex, N=2, scores 3.5 and 3.8):  
ProtocolRisk = 3.8 + (2-1) × 0.20 × 3.65 = 3.8 + 0.73 = **4.53**

*Note: The obligatorily linked protocol penalty (+0.5) has been removed. This formula scales naturally with N and subsumes it.*

The 0.20 coefficient was calibrated by testing across representative multi-protocol stacks (Curve+Convex, wstETH looping on Spark, RWA strategies) to ensure the penalty scales appropriately with compositional complexity.

**Multi-Protocol Hard Cap:**
- N is capped at 3. Strategies involving > 3 distinct protocols are automatically scored ProtocolRisk = 9.0 and flagged for CEO review.  
  Rationale: compositional complexity beyond 3 protocols introduces systemic dependency chains that cannot be adequately modeled by this formula.

**Output cap:** ProtocolRisk = min(formula result, 10.0)

#### 4.2.3 Multi-Asset Rule

When a strategy involves multiple assets:
- **Paired assets (LP pools):** AssetRisk = 0.75 × max(Ai) + 0.25 × mean(Ai)
- **Single-asset strategies:** AssetRisk = Asset score directly

**Example** (ynETHx score 7.15 + WETH score 1.0, mean = 4.075):  
AssetRisk = 0.75 × 7.15 + 0.25 × 4.075 = 5.36 + 1.02 = **6.38**

#### 4.2.4 Eligibility Thresholds

| GRS Range | Status |
|-----------|--------|
| ≤ 7.5 | APPROVED — eligible for vault deployment, subject to Section 5 rules |
| > 7.5 | EXCLUDED — ineligible |

#### 4.2.5 Missing Data Protocol

For the complete Missing Data Protocol (four cases including asset-specific Case 4 for <30-day assets), see `methodology/layer2_asset_assessment_methodology.md §11`. Summary:
- **Case 1 — Data unavailable but inferable:** Score at conservative midpoint of likely band + flag as LIMITATION
- **Case 2 — Data contradicts assessor expectation:** Cross-verify via ≥2 independent sources; if divergence >20%, use most conservative estimate
- **Case 3 — Data entirely missing on critical factor:** Score at band 9–10 (conservative MISSING) per §4.2.1 non-compensable principle
- **Case 4 (L2-specific) — Asset <30 days operational:** Defer full scoring; document pre-scoring baseline + re-assess at 30-day mark

---

### 4.3 Strategy-Specific Risk Factors by Type

#### 4.3.1 Type 1 — Looping / Leverage Strategies

Applicable to: recursive lending (e.g., deposit wstETH → borrow ETH → re-deposit on Spark/Vesu/IPOR)

**Strategy-Specific Risk** = 0.35·S1 + 0.25·S2 + 0.20·S3 + 0.20·S4

| # | Factor | Weight | Description |
|---|--------|--------|-------------|
| S1 | **Leverage-Adjusted Correlation Risk** | 35% | Correlation between collateral and borrowed asset under stress; amplified by leverage |
| S2 | **Stress Liquidation Buffer** | 25% | Buffer between current position and liquidation threshold under historical max depeg scenario |
| S3 | **Oracle Risk** | 20% | Oracle type, heartbeat, source count, circuit breaker presence |
| S4 | **Unwind Liquidity** | 20% | Ability to exit full position in stressed markets without excessive slippage |

**Scoring Rubrics:**

| Score | S1 — Leverage-Adjusted Correlation Risk |
|-------|-----------------------------------------|
| 1–2 | Highly correlated assets (wstETH/ETH, WBTC/tBTC), leverage ≤ 10x, max historical depeg < 5% |
| 3–4 | Highly correlated assets, leverage 10–15x, or max historical depeg 5–10% |
| 4–5 | Moderately correlated, leverage ≤ 5x |
| 6 | Moderately correlated, leverage 5–8x, or limited depeg history (< 12 months data) |
| 7–8 | Low correlation, leverage ≤ 3x |
| 9–10 | Uncorrelated assets, leverage > 3x |

| Score | S2 — Stress Liquidation Buffer |
|-------|-------------------------------|
| 1–2 | Under historical max depeg scenario, buffer > 30% above liquidation threshold |
| 3–4 | Buffer 20–30% above liquidation threshold under historical max depeg |
| 4–5 | Buffer 10–20% above liquidation threshold |
| 6 | Buffer 5–10% above liquidation threshold |
| 7–8 | Buffer < 5% above liquidation threshold |
| 9–10 | Would be liquidated under historical max depeg, or MISSING |

| Score | S3 — Oracle Risk |
|-------|-----------------|
| 1–2 | Chainlink, ≤ 1h heartbeat, 3+ sources, circuit breakers active |
| 3–4 | Chainlink or equivalent, 1–6h heartbeat, 3+ sources, no circuit breakers |
| 4–5 | Chainlink, 1–6h heartbeat, 2 sources |
| 6 | Mixed oracle setup (e.g., Chainlink + TWAP), or 6–24h heartbeat, or 2 sources without circuit breakers |
| 7–8 | Single oracle, TWAP-only, or > 24h heartbeat |
| 9–10 | Unverified oracle, or MISSING |

| Score | S4 — Unwind Liquidity |
|-------|----------------------|
| 1–2 | Can unwind full position in < 1h with < 1% slippage |
| 3–4 | Can unwind in 1–2h with 1–2% slippage; sufficient DEX depth across multiple venues |
| 4–5 | Can unwind in < 4h with < 2% slippage |
| 6 | Can unwind in 4–8h with 2–5% slippage; fragmented liquidity across venues |
| 7–8 | Unwind takes > 4h or > 5% slippage |
| 9–10 | Cannot unwind within 24h, or MISSING |

---

#### 4.3.2 Type 2A — LP / AMM Classic Strategies

Applicable to: providing liquidity in Curve, Balancer, or Uniswap pools, with or without Convex/Aura boosting

**Strategy-Specific Risk** = 0.35·S1 + 0.30·S2 + 0.20·S3 + 0.15·S4

| # | Factor | Weight | Description |
|---|--------|--------|-------------|
| S1 | **IL Risk** | 35% | Expected impermanent loss based on asset correlation and volatility |
| S2 | **Depeg Correlation** | 30% | Probability of paired assets depegging simultaneously |
| S3 | **Dual Contract Surface** | 20% | Additional smart contract risk from pool + booster/wrapper |
| S4 | **Reward Token Risk** | 15% | Volatility and liquidity of reward tokens (CRV, CVX, etc.) |

**Scoring Rubrics:**

| Score | S1 — IL Risk |
|-------|-------------|
| 1–2 | Stablecoins or highly correlated LSTs — IL < 0.1% historical |
| 3–4 | Correlated LST pairs with minor basis risk — IL 0.1–0.5% historical |
| 4–5 | Moderately correlated LSTs — IL 0.5–2% |
| 6 | LST vs. stablecoin, or moderate correlation with known volatility — IL 2–3% |
| 7–8 | Volatile vs. stable, or LST vs. non-LST — IL 2–5% |
| 9–10 | Uncorrelated assets — IL > 5% |

| Score | S2 — Depeg Correlation |
|-------|----------------------|
| 1–2 | Independent peg mechanisms, no common dependency |
| 3–4 | Low depeg correlation — different issuers or mechanisms, minor shared systemic exposure |
| 4–5 | Partial dependency (same underlying protocol) |
| 6 | Moderate-to-high shared dependency — significant common collateral or counterparty exposure |
| 7–8 | Strong systemic dependency — if one depegs, the other likely follows |
| 9–10 | Total dependency, or MISSING |

| Score | S3 — Dual Contract Surface |
|-------|--------------------------|
| 1–2 | Pool + booster audited 3+ times by Tier 1 firms, operational > 2 years, zero incidents |
| 3–4 | Pool + booster audited 2+ times by reputable firms, operational > 2 years, or no booster layer (minimal wrapper) |
| 4–5 | Pool + booster audited 2 times, operational > 1 year |
| 6 | Pool audited; booster audited once by Tier 2 firm only, or both audited once with operational history 1–2 years and no incidents |
| 7–8 | Pool or booster audited once only, or operational < 1 year |
| 9–10 | Pool or booster unaudited, or MISSING |

| Score | S4 — Reward Token Risk |
|-------|----------------------|
| 1–2 | No reward token (fees only), or reward in ETH/USDC/BTC |
| 3–4 | CRV, CVX — top-50 market cap, deep liquidity |
| 5–6 | Mid-cap governance token (top 100–200), moderate liquidity, meaningful emissions but established market |
| 7–8 | Small-cap token, low liquidity, high emissions relative to market cap |
| 9–10 | Unknown token, farm-and-dump dynamics, or MISSING |

**Concentrated-liquidity LP edge case** (Uniswap V3, V4, Bunni, Algebra, Trader Joe Liquidity Book):

The S1 IL Risk rubric above assumes constant-product or stableswap bonding curves. Concentrated-liquidity LP positions have fundamentally different IL behavior — out-of-range positions earn zero fees and accumulate IL asymmetrically as price moves through the band.

- **Stable-stable concentrated LP** (e.g., USDC/USDT 0.01% on Uniswap V3 with tight band): apply standard Type 2A scoring; S1 IL Risk lands 1–3 normally.
- **Volatile-stable or volatile-volatile concentrated LP** (e.g., ETH/USDC 0.05%, BTC/ETH): apply **S1 ≥ 6 floor** to capture range-management risk. Active-management protocols that auto-rebalance ranges (Arrakis, Gamma, Bunni LIT, Steer) reduce this to **S1 ≥ 4 floor**; passive concentrated LPs retain S1 ≥ 6.
- S4 sub-modifier: most concentrated-LP positions earn fees only (no governance emissions) → S4 typically 1–2 unless paired with a third-party booster.

---

#### 4.3.3 Type 2B — LP Pendle Strategies

Applicable to: providing liquidity in Pendle AMM pools (PT/SY or PT/YT splits of yield-bearing tokens)

**Strategy-Specific Risk** = 0.35·S1 + 0.30·S2 + 0.20·S3 + 0.15·S4

| # | Factor | Weight | Description |
|---|--------|--------|-------------|
| S1 | **Yield Divergence Risk** | 35% | Volatility and predictability of the underlying yield rate |
| S2 | **Underlying Asset Risk** | 30% | Mapped directly from Layer 2 Asset Risk Score |
| S3 | **Maturity Risk** | 20% | Time to pool expiry; liquidity and yield exposure deteriorate near maturity |
| S4 | **Pool Liquidity** | 15% | Pool TVL depth and slippage for institutional-size exits |

**Scoring Rubrics:**

| Score | S1 — Yield Divergence Risk |
|-------|--------------------------|
| 1–2 | Very stable and predictable underlying yield (ETH staking ~3–4%), yield vol < 10% annualized |
| 3–4 | Mostly stable yield with minor market-rate sensitivity (e.g., LST-based lending rates), yield vol 10–20% annualized |
| 4–5 | Moderately variable yield (mixed LST strategies), yield vol 20–50% annualized |
| 6 | Variable yield with meaningful exposure to protocol incentives or market conditions, yield vol 30–50% annualized |
| 7–8 | Highly variable yield (funding rate, junior tranche), yield vol > 50% |
| 9–10 | Unpredictable yield, or MISSING |

**S2 — Underlying Asset Risk mapping (Layer 2 → S2 score):**

| Layer 2 Asset Score | S2 Score |
|---------------------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 5.0 | 4 |
| 5.0 – 6.5 | 6 |
| 6.5 – 7.5 | 8 |
| > 7.5 | 10 |

| Score | S3 — Maturity Risk |
|-------|------------------|
| 1–2 | Maturity > 6 months — low yield exposure, stable position |
| 3–4 | Maturity 4–6 months — comfortable runway, limited yield exposure, ample time to roll |
| 5–6 | Maturity 2–4 months — moderate-to-elevated exposure, pool depth beginning to decline |
| 7–8 | Maturity < 2 months — maximum yield exposure, pool may become illiquid near expiry |
| 9–10 | Maturity < 2 weeks, or MISSING |

| Score | S4 — Pool Liquidity |
|-------|-------------------|
| 1–2 | Pool TVL > $50M; $1M exit with < 0.5% slippage |
| 3–4 | Pool TVL $25M–$50M; $1M exit with 0.25–0.5% slippage |
| 4–5 | Pool TVL $10M–$50M; $1M exit with 0.5–2% slippage |
| 6 | Pool TVL $5M–$10M; $1M exit with 1–3% slippage |
| 7–8 | Pool TVL < $10M; $1M exit with > 2% slippage |
| 9–10 | Essentially illiquid, or MISSING |

---

#### 4.3.4 Type 2C — Pendle PT (Bond-like) Strategies

Applicable to: holding standalone Pendle Principal Tokens (PT) without LP exposure. The user buys PT at a discount, holds to maturity (redeem 1:1 for the underlying yield-bearing asset), or exits early via Pendle's PT/SY orderbook. **No IL, no AMM bonding-curve exposure** — the risk profile is bond-like, dominated by the underlying asset's solvency through maturity.

**Strategy-Specific Risk** = 0.40·S1 + 0.30·S2 + 0.20·S3 + 0.10·S4

| # | Factor | Weight | Description |
|---|--------|--------|-------------|
| S1 | **Underlying Asset Risk** | 40% | Mapped directly from Layer 2 ARS of the underlying yield-bearing asset (the asset PT redeems into at maturity) |
| S2 | **Maturity Risk** | 30% | Time to expiration; near-maturity escalation pattern |
| S3 | **Yield Source Quality** | 20% | Survivability of the embedded yield mechanism through maturity (does the underlying impair before redemption?) |
| S4 | **PT Exit Liquidity** | 10% | Depth of Pendle PT/SY orderbook for early-exit at institutional size; can be moderated −1 band if hold-to-maturity is the explicit plan |

**Scoring Rubrics:**

**S1 — Underlying Asset Risk** (mapped from L2 ARS — same table as Type 2B S2):

| Layer 2 Asset Score | S1 Score |
|---------------------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 5.0 | 4 |
| 5.0 – 6.5 | 6 |
| 6.5 – 7.5 | 8 |
| > 7.5 | 10 |

**Cascade rule:** If underlying L2 ARS > 6.5 (v3.7), **strategy is EXCLUDED** via §4.2.1 L2 hard cap. Do not compute S1; document the cascade. Assets in WATCHLIST band (ARS 6.0–6.5) trigger §5 Rule 3 concentration cap, not exclusion.

**S2 — Maturity Risk** (same rubric as Type 2B S3):

| Score | Criteria |
|-------|----------|
| 1–2 | Maturity > 6 months — fixed-yield horizon comfortable, redemption far-dated |
| 3–4 | Maturity 4–6 months — comfortable runway |
| 5–6 | Maturity 2–4 months — moderate operational attention |
| 7–8 | Maturity < 2 months — near-expiry, plan exit or roll |
| 9–10 | Maturity < 2 weeks, or MISSING |

**S3 — Yield Source Quality** (NEW — measures whether the embedded yield mechanism can break before maturity, impairing PT redemption):

| Score | Criteria |
|-------|----------|
| 1–2 | RWA T-bill backing, ETH staking — extremely stable embedded yield, near-zero failure risk pre-maturity |
| 3–4 | Lending rate (Aave/Morpho/Spark supply rate) — stable but rate-dependent; mechanism cannot fail |
| 5–6 | Hybrid mechanism (e.g., LST-based lending), or moderate yield source variability |
| 7–8 | Funding-rate-dependent (Ethena sUSDe, BasisOS) — yield can collapse but PT redemption ratio survives unless backing impaired |
| 9–10 | First-loss tranche, junior position, or experimental/novel yield source — material risk that the underlying is impaired before maturity, breaking PT redemption |

**S4 — PT Exit Liquidity:**

| Score | Criteria |
|-------|----------|
| 1–2 | PT/SY pool TVL > $50M; $1M PT exit < 0.5% slippage |
| 3–4 | PT/SY pool TVL $25–50M; $1M exit 0.5–1% slippage |
| 4–5 | PT/SY pool TVL $10–25M; $1M exit 1–2% slippage |
| 6 | PT/SY pool TVL $5–10M; $1M exit 2–3% slippage |
| 7–8 | PT/SY pool TVL < $5M |
| 9–10 | No active PT/SY orderbook (must hold to maturity), or MISSING |

**Hold-to-maturity moderator:** If the deployment plan is explicitly "buy and hold to maturity, no early exit needed", S4 may be moderated by 1 band downward (since exit liquidity is not a binding constraint). Document the hold-to-maturity intent in the assessment.

#### 4.3.5 Type 3 — Lending Strategies

Applicable to: supplying assets to a lending position with **1 or more underlying markets**:
- **N=1 (direct lending):** deposit into a single isolated lending market (e.g., a Morpho Blue isolated market, a Felix isolated market, a single Euler vault) — no curator, parameters fixed at market creation
- **N≥2 (curated vault):** deposit into a curated lending vault that allocates across multiple markets (e.g., MetaMorpho, Euler curated, Yearn V3, Ipor)

The same S-factor framework applies to both; S1 is evaluated under the rubric matching N.

**Strategy-Specific Risk** = 0.30·S1 + 0.25·S2 + 0.20·S3 + 0.15·S4 + 0.10·S5

| # | Factor | Weight | Description |
|---|--------|--------|-------------|
| S1 | **Risk Management Quality** | 30% | N=1: market configuration quality (LLTV, oracle, IRM, creator). N≥2: curator track record, audit status, transparency. |
| S2 | **Collateral Composition** | 25% | Weighted average Layer 2 risk score of underlying collateral assets (single asset for N=1) |
| S3 | **Utilization Spike Risk** | 20% | Historical frequency and duration of > 90% utilization episodes |
| S4 | **Collateral Correlation Risk** | 15% | Concentration in correlated collateral assets (single-collateral N=1 markets land 7–8 by structure) |
| S5 | **Bad Debt History** | 10% | Historical bad debt as a proportion of TVL |

**Scoring Rubrics:**

**S1 — Risk Management Quality** — apply the rubric matching the strategy's N:

**Mode A — Direct single-market (N=1):** Score the static parameters set at market creation. For Morpho Blue and similar protocols, market parameters are immutable post-creation, so this scoring is permanent.

| Score | S1 (Mode A — Direct N=1) |
|-------|--------------------------|
| 1–2 | Conservative LLTV (≥10pp buffer below comparable mainstream markets for the collateral risk band), Tier-1 oracle (Chainlink/Pyth) with multi-source + circuit breaker, battle-tested IRM (e.g., Morpho AdaptiveCurveIRM), market creator is a doxxed reputable team coordinating with the collateral issuer |
| 3–4 | Reasonable LLTV (5–10pp buffer), Tier-1 oracle (single-source acceptable), battle-tested IRM, market creator known and identifiable |
| 4–5 | LLTV typical for collateral risk band but with little buffer, mixed-tier oracle (single-source Chainlink, or Pyth+TWAP fallback), known IRM, market creator semi-known |
| 6 | LLTV at the aggressive end for the collateral's ARS, oracle adequate but lacking circuit breaker or fallback, IRM known, market creator pseudonymous-but-consistent |
| 7–8 | Aggressive LLTV vs. collateral ARS, custom or untested oracle, novel IRM, anonymous market creator |
| 9–10 | LLTV inappropriate for collateral risk, unverified oracle, untested IRM, market created by unverifiable counterparty, or MISSING |

**Mode B — Curated vault (N≥2):** Score the curator.

| Score | S1 (Mode B — Curated N≥2) |
|-------|---------------------------|
| 1–2 | Tier 1 curator (Gauntlet, Steakhouse, Re7, B.Protocol) — track record > 1 year, audited processes, public decision logs |
| 3–4 | Tier 1 curator with track record < 1 year, or Tier 2 curator with detailed public risk reports and audit coverage of processes |
| 4–5 | Tier 2 curator — known team, less track record |
| 6 | Semi-known team with public presence, no formal process audit, limited or inconsistent decision logs |
| 7–8 | Unknown or unverified curator |
| 9–10 | No curator / unmanaged vault, or MISSING |

**S2 — Collateral Composition mapping (weighted avg Layer 2 score of vault collateral):**

| Weighted Avg Layer 2 Score | S2 Score |
|---------------------------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 4.5 | 4 |
| 4.5 – 6.0 | 6 |
| > 6.0 | 8 |
| Unknown collateral | 9 |

| Score | S3 — Utilization Spike Risk |
|-------|---------------------------|
| 1–2 | Utilization never exceeded 90% over 90 days, low volatility |
| 3–4 | 1 episode > 90% over 90 days, duration < 1h, resolved without user impact |
| 4–5 | 1–2 episodes > 90% over 90 days, duration < 4h each |
| 6 | 2–3 episodes > 90% over 90 days, some exceeding 4h but < 12h each |
| 7–8 | Frequent episodes > 90%, or any episode > 24h |
| 9–10 | MISSING — historical data unavailable |

| Score | S4 — Collateral Correlation Risk |
|-------|--------------------------------|
| 1–2 | Markets exposed to 3+ different assets with low correlation |
| 3–4 | Markets exposed to 2–3 asset classes, low-to-moderate correlation, no single dominant exposure |
| 4–5 | 2 dominant assets, moderate correlation |
| 6 | 2 dominant assets with meaningful shared systemic risk (e.g., common collateral type or issuer) |
| 7–8 | Single collateral asset, or very high correlation across markets |
| 9–10 | MISSING |

| Score | S5 — Bad Debt History |
|-------|----------------------|
| 1–2 | Zero bad debt, vault operational > 6 months |
| 3–4 | Zero bad debt, vault operational 3–6 months |
| 4–5 | Bad debt < 0.01% of TVL, resolved quickly |
| 6 | Bad debt < 0.01% of TVL, but resolution took > 7 days or required curator intervention |
| 7–8 | Bad debt > 0.01%, or unresolved incidents |
| 9–10 | MISSING, or vault < 3 months with no history |

---

## 5. Vault-Level Aggregation

Each ForgeYields vault (fyUSDC, fyETH, fyWBTC) may deploy across multiple strategies. The Vault Risk Score is:

**Vault Risk Score** = Σ (w_i · GRS_i)

Where w_i is the capital allocation weight of strategy i in the vault.

---

### Rule 1 — Vault Risk Score cap

Vault Risk Score must remain ≤ 5.5 at all times.

The 5.5 cap reflects ForgeYields' risk appetite: higher than conservative treasury managers (3.0–4.0 range) but below aggressive yield farms (6.0+ range).

---

### Rule 2 — Protocol concentration cap

| Protocol Risk Score | Max Allocation in Vault |
|--------------------|------------------------|
| 1.0 – 2.5 | No limit |
| 2.6 – 4.0 | 50% |
| 4.1 – 5.5 | 30% |
| 5.6 – 7.0 | 15% |
| > 7.0 | Not eligible |

---

### Rule 3 — Asset concentration cap

| Asset Risk Score | Max Exposure in Vault | Status |
|-----------------|----------------------|--------|
| 1.0 – 3.0 | No limit | APPROVED |
| 3.1 – 5.0 | 60% | APPROVED |
| 5.1 – 6.0 | 30% | APPROVED |
| **6.0 – 6.5** | **15%** | **WATCHLIST** *(v3.7)* |
| > 6.5 | Not eligible | EXCLUDED |

Exposure is calculated as the sum of allocations across all strategies that use this asset as primary collateral or underlying.

**WATCHLIST band (v3.7):** Assets with ARS in the 6.0–6.5 range are eligible for deployment with a 15% per-vault concentration cap. They require:
- Monthly re-scoring cadence (vs. quarterly for standard APPROVED)
- Documented monitoring trigger list in the L2 assessment file (specific events that would force re-evaluation: NAV deviation thresholds, mechanism pause events, mcap drops, etc.)
- Quarterly CEO review with documented hold/exit decision
- Position sizing within Type-specific Rule 4 caps still applies and may bind tighter than the 15% Rule 3 cap.

---

### Rule 4 — Position sizing cap per strategy type

**Type 1 — Looping:**
- ForgeYields borrow amount ≤ 20% of total market borrow
- DEX liquidity at 2% depth ≥ 2× ForgeYields borrow amount

**Type 2A — LP/AMM Classic:**
- ForgeYields position ≤ 10% of pool TVL

**Type 2B — LP Pendle:**
- ForgeYields position ≤ 15% of pool TVL
- ForgeYields position ≤ pool AMM liquidity / 2

**Type 2C — Pendle PT (Bond-like):**
- ForgeYields position ≤ 20% of total PT supply at the relevant maturity
- ForgeYields position ≤ 20% of vault TVL per individual maturity
- Aggregate across all PT positions on the same underlying asset ≤ 30% of vault TVL (prevents stacking the same underlying across multiple maturities)

**Type 3 — Lending:**

| S2 Collateral Composition Score | Max Position in Vault |
|--------------------------------|----------------------|
| 1–2 | 25% of vault TVL |
| 3–4 | 20% of vault TVL |
| 5–6 | 15% of vault TVL |
| 7–8 | 10% of vault TVL |

Protocol and asset concentration caps (Rules 2–3) provide indirect correlation risk management. A full correlation matrix is not implemented at this stage but may be added as vault complexity increases.

---

## 6. Governance and Review Cadence

| Activity                    | Frequency          |
| --------------------------- | ------------------ |
| Full Layer 1 re-scoring     | Quarterly          |
| Full Layer 2 re-scoring     | Monthly            |
| Layer 3 strategy review     | On material change (governance rule modification, major version upgrade, new audit findings rated medium or higher, or TVL delta >20% within 7 days) |
| Methodology document review | Semi-annually      |

---

## 7. Changelog

| Version | Date | Changes |
|---------|------|---------|
| 1.0 | 2026-04-03 | Initial release |
| 2.0 | 2026-04-03 | Added C6 (Team & Transparency), split C4 into incident history + operational age, adjusted Layer 1 weights. Added A5 (Market Cap / Supply Concentration), RWA override for A4, yield-bearing stablecoin note, adjusted Layer 2 weights. Updated composite formula to 0.35/0.25/0.40, added linked-protocol penalty. Updated worked examples. |
| 2.1 | 2026-04-09 | Added Type 2B (LP Pendle) and Type 3 (Lending Vault) strategy types. Replaced Section 7 worked examples with live ForgeYields strategies (fyETH-001: spark-wstETH-WETH-looping, GRS 2.59; fyUSDC-011: gteUSDc-morpho-gauntlet, GRS 2.42). Replaced fixed 40% single-strategy allocation cap with registry-driven per-strategy caps in Section 5. |
| 2.2 | 2026-04-09 | Removed Section 7 (Worked Examples) and Section 9 (Appendix — Score Summary Table Template). Renumbered: Section 8 (Governance and Review Cadence) → Section 7; Section 10 (Changelog) → Section 8. Worked examples are now maintained exclusively in strategy_registry.json. |
| 2.3 | 2026-04-09 | Replaced Section 5 with structured vault-level aggregation rules: Rule 1 (Vault Risk Score cap ≤ 5.5), Rule 2 (protocol concentration cap table by protocol risk score), Rule 3 (asset concentration cap table by asset risk score), Rule 4 (position sizing caps per strategy type — Looping, LP/AMM Classic, LP Pendle, Lending Vault). Formula updated from StrategyRisk to GRS notation. |
| 2.4 | 2026-04-09 | Removed Section 6 (Monitoring and Rebalancing Triggers). Renumbered: Section 7 (Governance and Review Cadence) → Section 6; Section 8 (Changelog) → Section 7. |
| 2.5 | 2026-04-09 | Simplified Section 4.2.4 (Eligibility Thresholds) to two tiers: ≤ 7.5 = APPROVED, > 7.5 = EXCLUDED. Removed WATCHLIST tier. Replaced all remaining WATCHLIST references in the document with APPROVED. |
| 2.6 | 2026-04-09 | Replaced Section 4.2.2 (Multi-Protocol Rule) with scaling formula: ProtocolRisk = max(Pi) + (N-1) × 0.20 × mean(Pi). Removed obligatorily linked protocol flat +0.5 penalty — subsumed by the new formula. Replaced Section 4.2.3 (Multi-Asset Rule) with: paired assets AssetRisk = 0.75 × max(Ai) + 0.25 × mean(Ai), replacing previous 0.5/0.5 split. Added worked examples to both sections. |
| 2.7 | 2026-04-09 | Replaced Section 4.2.1 (Weakest Link Rules) with a single unified Weakest Link Rule: any protocol score > 7.0 or any asset score > 6.0 triggers immediate EXCLUDED status regardless of GRS, with mandatory wind-down if already deployed. Removed the erroneous four-row table that incorrectly labeled scores > 7.0 as APPROVED. Added cross-reference note to Section 5 Rules 2 and 3. |
| 2.8 | 2026-04-10 | (1) Section 4.2.1: clarified Weakest Link Rule — replaces "mandatory wind-down if already deployed" with explicit 7-day exit window for already-deployed strategies. (2) Section 4.2.2: added calibration note for 0.20 coefficient. (3) Section 5 Rule 4 Type 1: tightened DEX liquidity criterion to "at 2% depth". (4) Sections 2.4 and 3.4: added data-source divergence rule (>20% triggers manual verification, most conservative estimate used). (5) Section 6: expanded "On material change" definition for Layer 3 review cadence. (6) Section 2.1: added bridge risk scoping note (canonical vs. third-party bridges, CCTP, LayerZero DVN). (7) Section 5 Rule 1: added context note on 5.5 cap relative to market benchmarks. (8) Section 5: added correlation risk management disclaimer after Rule 4. |
| 2.9 | 2026-04-10 | Section 1 Executive Summary: replaced eligibility text — removed watchlist tier (6.1–7.5) and ≤ 6.0 cap; new language: strategies scoring ≤ 7.5 are eligible subject to Section 5 concentration and position sizing rules; strategies scoring > 7.5 are excluded. |
| 3.0 | 2026-04-10 | **Breaking change:** Added per-criterion hard caps to Weakest Link Rule (Section 4.2.1). C3, C4, C6 ≥ 8 now trigger immediate EXCLUDED status regardless of PRS. Rationale: human trustworthiness factors (governance, incident history, team transparency) are non-compensable by technical excellence. Validated empirically via Resolv pre-exploit backtest (C3=9, PRS=5.32 → would now be correctly EXCLUDED). Criteria C1, C2, C5 remain weighted-average-only (no hard cap). All protocols in registry re-scanned; any with C3/C4/C6 ≥ 8 updated to status: excluded. |
| 3.1 | 2026-04-10 | (1) Section 2.3 C4: added explicit scoring process instruction before C4a rubric — "Score C4a and C4b independently, then combine: C4 = 0.70 × C4a + 0.30 × C4b". (2) Section 2.3 C3: added operational security note clarifying that key management practices (HSM, rotation, access controls) are assessed qualitatively and may trigger exclusion regardless of C3 score. (3) Section 2.3 C5: expanded 7-8 row with off-chain infrastructure scoring note — protocols critically dependent on CEX accounts or cloud services for core operations score minimum 7. (4) Section 4.2.1: replaced Weakest Link Rule — added hard caps C1 ≥ 9 and C5 ≥ 9 (fundamental technical risk, non-compensable); added asset-level hard cap A2 ≥ 9 (sustained depeg > 7 days indicates mechanism failure); expanded rationale to distinguish human counterparty risk from fundamental technical risk. |
| 3.2 | 2026-04-13 | Section 4.2.2 (Multi-Protocol Rule): added two hard caps after the calibration note. (1) N cap: N is capped at 3; strategies with > 3 protocols are automatically scored ProtocolRisk = 9.0 and flagged for CEO review (rationale: compositional complexity beyond 3 protocols introduces systemic dependency chains not adequately modeled by the formula). (2) Output cap: ProtocolRisk = min(formula result, 10.0) to prevent scores exceeding the 10-point scale. |
| 3.3 | 2026-04-13 | Section 2.3: added rubric vs. operational proxy distinction note at the top of the scoring rubrics section — rubrics define qualitative score band meaning; layer-specific methodology files define operational proxies (e.g., LOC thresholds for C5); proxies do not override rubrics; rubric definitions take precedence in case of conflict. Corresponding label added in layer1_protocol_assessment_methodology.md C5 complexity indicators. |
| 3.4 | 2026-04-14 | Section 2.3 C1 (Audit Status): added upgradeability pre-scoring step and split rubric into Type A (immutable core, audit recency not penalized) and Type B (upgradeable core, full recency rubric). Introduced Unaudited Code Delta (UCD) concept for Type A scoring. Resolves systematic over-penalization of immutable protocols with aging audits. Triggered by FORA-136 (Convex Finance C1=7 despite immutable core). Corresponding changes in layer1_protocol_assessment_methodology.md C1 section (upgradeability pre-check, Step 1b, Step 3b). Convex Finance re-scored under Type A: C1 7→5, PRS 5.89→5.39. |
| 3.5 | 2026-04-23 | §4.2.1: Replaced C4 hard cap `C4 ≥ 8` with `max(C4a, C4 composite) ≥ 8`. Prevents catastrophic incident history (C4a ≥ 8) from being diluted by operational age (C4b) under the composite formula. Empirically validated by Kelp DAO $292M Lazarus exploit (pre-incident C4 composite = 7.50 failed to trigger v3.4 rule despite C4a = 9). Four currently-approved L1 protocols flipped to EXCLUDED on activation: Balancer V2, Renzo, Usual Protocol, Inverse Finance FiRM. USD0 L2 cascaded to L1-cascaded-excluded (like apxUSD pattern). See `methodology/v35_amendment.md`. |
| 3.6 | 2026-04-23 | §4.2.1: Added C5 cross-chain configuration hard cap — 1-of-1 DVN / single verifier → C5 ≥ 9 (exclusion); bridge threshold < 2-of-N → C5 ≥ 8 (watchlist); undisclosed configuration after mandatory verification → C5 ≥ 8. Added C1 audit coverage gap floors — UCD > 50% → C1 ≥ 7; major production layer entirely unaudited → C1 ≥ 8; acknowledged critical/high finding > 90 days unfixed → C1 ≥ 7. Added Rule C (operational) — public security warnings from recognized firms trigger mandatory 30-day re-assessment. Validated via Kelp 1-of-1 DVN (pre-incident score C5 = 7 → v3.6 would force C5 = 9 → exclusion). Verification phase (on-chain DVN reads) confirmed: Ethena compliant (4-of-4 DVN), Bedrock watchlist (Celer cBridge, C5 5→8), Solv watchlist (FROST audit gap, C1 4→8). Zero false-positive flips. See `methodology/v36_amendment.md`. |
| **3.9** | **2026-05-07** | **L1 C3 Custody Mode Recognition.** Closes gap exposed by Saturn Protocol's response to its EXCLUDED verdict (disclosed Fireblocks MPC 2/3 custody on the EOA holding DEFAULT_ADMIN_ROLE). Pre-v3.9 strict reading treated all single on-chain EOAs identically, regardless of off-chain custody mode (plain hot wallet vs HSM vs MPC threshold-signature). v3.9 introduces 3-tier framework: (Tier A) plain EOA + HSM/AWS-KMS single-key remain C3=9 EXCLUDED unchanged (Resolv pattern protected); (Tier B) MPC threshold ≥2/3 with verifiable custodian attestation can mitigate to C3=7-8 qualitatively (off-chain trust, opacity penalty); (Tier C) on-chain Timelock + multisig provides full mitigation to C3=4-6. Default reading remains strict — undisclosed custody → C3=9. Attestation requirements specified: custodian-signed letter (Fireblocks/Copper/BitGo Trust/Anchorage/Coinbase Custody Tier 1, or Tier 2/unknown), Policy Engine config, customer attestation with 30-day no-change clause, 6-month staleness limit. Tier B never drops below C3=7 without on-chain Timelock. Forward-looking framework — Saturn/Noon/Reservoir EXCLUDED status preserved pending attestation collection. Outreach template provided for active re-assessment of EXCLUDED protocols. See `methodology/v39_amendment.md`. |\n| **3.8** | **2026-05-07** | **L3 Recursive Strategy Collateral Rule.** Closes the gap where Type 3 lending strategies use a Pendle PT/YT/LP, AMM LP, or non-L2 lending receipt token as collateral. Rule applies (1) Multi-Protocol Rule N+1 inflation (count the L3 strategy issuer protocol as additional N); (2) S2 mapping uses underlying fundamental L2 ARS, NOT a derived synthetic from L3 GRS; (3) S2 floor of 4 to recognize recursive composition complexity even with high-quality underlying; (4) S1 +1 band penalty to capture recursive oracle/mechanism risk; (5) cascade-EXCLUDE if underlying L3 strategy is EXCLUDED. Forward-looking — zero retroactive impact on current registry. Conservative-by-design: tail-risk pricing maintained via S1+1 + S2 floor + N+1 protocol counting; no hard cap relaxation. See `methodology/v38_amendment.md`. |\n| **3.7** | **2026-05-06** | **L2 dual amendment: A3 async exit credit + WATCHLIST band 6.0–6.5.** (1) §3.3 A3: added formal "Async Exit Credit" modifier — atomic redeem (SLA=0) gets −2 bands; queue/cooldown SLA ≤ 7 days gets −1 band; SLA ≤ 14 days gets −1 band conditional; SLA > 14 days or pause history → no credit. Floor at band 3, max −2 bands total. Eligibility requires documented mechanism + on-chain verifiable contract + no pause >SLA in past 12 months. Rationale: corrects asymmetric application where DEX-deep assets got implicit queue credit while DEX-thin assets did not, despite identical functional exit capability for patient holders. (2) §4.2.1 + §5 Rule 3: ARS hard cap raised from > 6.0 to > 6.5; new WATCHLIST band 6.0–6.5 with 15% per-vault concentration cap. Rationale: combinatorial scoring (multiple mid-band penalties) was over-clustering at 6.0–6.5 even when no single criterion hit band 9+; watchlist band provides graduated exposure with mandatory monthly re-scoring + documented monitoring triggers. Aligns L2 cap structure with L1 (PRS > 7 watchlist) and L3 (GRS > 7.5 hard cap) which already have intermediate bands. Audit pass on registry 2026-05-06: ynETHx (6.60 → 6.35 with A3 credit) → WATCHLIST 15%; ynRWAx (6.20, no async path) → WATCHLIST 15% subject to RWA legal-enforceability override review; syrupUSDC (4.45 → 4.20 with queue credit) → APPROVED unchanged cap. xsBTC (6.95), jrUSDe (6.75), triBTC (6.70), apyUSD (6.60 + L1 cascade) → confirmed EXCLUDED (>6.5). Asset count impact: +2 assets to WATCHLIST tier, no APPROVED tier additions, no false-positive flips. |

---

*This methodology is maintained by the ForgeYields Risk Analyst and is subject to revision based on market conditions, new protocol integrations, and feedback from institutional LPs.*
