# Layer 3 Strategy Assessment — Field Guide

**Current methodology version:** v4.2 (May 2026)
**Last L3-specific amendments:** v3.8 (Recursive Strategy Collateral Rule, 2026-05-07) · v4.1 (Type W Wrapper Vault X1–X5 rubric, 2026-05-18)
**L3 core structure:** unchanged since v3.4. Cascade gate (§0.2.3) inherits L1/L2 hard caps including v3.5/v3.6 L1 changes and **v3.7 L2 ARS > 6.5 cap with 6.0–6.5 WATCHLIST band**. Section 5W (Type W Wrapper Vault) added in v4.1.
**Date:** Last meaningful edit 2026-05-18 (v4.1).
**Companion to:** `full-framework.md` (core rubrics), `layer1_protocol_assessment_methodology.md`, `layer2_asset_assessment_methodology.md`. See `amendments/` for v3.7, v3.8, v4.1 full text.

**Note on amendment applicability:** v3.5 and v3.6 are L1-focused rule changes. L3 inherits them automatically through the §0.2.3 cascade gate — any L1 hard cap trigger (including v3.5 `max(C4a, C4) ≥ 8` and v3.6 C5 cross-chain / C1 audit coverage rules) propagates to L3 exclusion.

TRIGGER: When explicitly instructed to assess a specific yield strategy using Layer 3 methodology. Do not trigger on general scoring requests — those are dispatched via `/fy-risk-assessment`.

---

## 0. Pre-scoring — Classification and Dependency Validation

Before computing any S-factor, complete these pre-scoring steps in order. A strategy cannot be meaningfully scored without them.

### 0.1 Strategy type classification (pick exactly one)

Use this decision tree:

```
1. Does the strategy involve borrowing against deposited collateral to re-deposit?
   YES  → Type 1 (Looping / Leverage)
   NO   → continue

2. Does the strategy hold standalone Pendle PT (Principal Token) without LP exposure?
   YES  → Type 2C (Pendle PT — Bond-like) — buy PT, hold to maturity or exit via PT/SY orderbook; no IL, no AMM exposure
   NO   → continue

3. Does the strategy provide liquidity to an AMM pool?
   YES  → Is the pool a Pendle AMM (PT/SY or PT/YT)?
          YES → Type 2B (LP Pendle)
          NO  → Type 2A (LP / AMM Classic — Curve, Balancer, Uniswap, Ekubo, Velodrome, Aerodrome, etc.)
   NO   → continue

4. Is the strategy a deposit into a lending market or curated lending vault (1+ underlying markets)?
   YES  → Type 3 (Lending) — N=1 for direct single-market deposit, N≥2 for curated multi-market vault
   NO   → STOP. Strategy is out of scope — flag to CEO for methodology extension.
```

**Hybrid strategies** (e.g., "loop into a Curve LP then deposit into Convex"):
- Classify by the **outermost user-facing action**: if users LP in → Type 2A; if users loop → Type 1
- Inner legs contribute via Multi-Protocol Rule (§3) and Multi-Asset Rule (§4)

### 0.2 Eligibility gate — L1 and L2 cascade validation (MANDATORY)

**Before scoring ANY S-factors**, verify all participating protocols and assets are eligible.

#### 0.2.1 Protocol cascade

For each protocol Pi in the strategy:
- Lookup in `registries/protocol_registry.json`
- If `verdict == "excluded"` OR `hardCapTrigger` is set → **Strategy is EXCLUDED.** Do not score; document the cascade path.
- If `verdict == "approved" && forgeYieldsIntegrated == true` but registry shows contradictory state (like `verdict: excluded && forgeYieldsIntegrated: true`) → **FLAG TO CEO** (stale integration predating current verdict); treat as EXCLUDED pending reconciliation.
- If protocol is **NOT IN REGISTRY** → **Strategy is BLOCKED pending L1 assessment.** Do not score; flag the missing L1.

#### 0.2.2 Asset cascade

For each asset Aj in the strategy:
- Lookup in `registries/asset_registry.json`
- If `eligibility == "excluded"` → **Strategy is EXCLUDED.** Document the cascade path.
- If `eligibility_subtype == "l1_cascaded (L2 standalone APPROVED; recoverable via L1 governance reform)"` → **Strategy is EXCLUDED pending L1 fix.** Note this is recoverable unlike pure L2 exclusions.
- If asset is **NOT IN REGISTRY** → **Strategy is BLOCKED pending L2 assessment.** Do not score.

#### 0.2.3 L3 Weakest Link Rule application

Per methodology §4.2.1, L3 inherits L1 and L2 hard caps. **ANY single L1 hard cap trigger (PRS>7, C1≥9, max(C4a, C4)≥8 [v3.5], C3≥8, C5≥9 [incl. v3.6 cross-chain 1-of-1 DVN], C6≥8) OR L2 hard cap trigger (ARS>6.5 [v3.7], A2≥9) on ANY participating protocol or asset excludes the strategy.**

If cascade excludes the strategy, the assessment output remains **fully computed** (per v4.2: GRS is always derived; verdict is a separate field). Required sections:
- Summary header
- §0 (classification + cascade table showing which trigger fired)
- §5–§8 S-factor scoring completed in full (supports continuous measurement and same-cycle reactivation if the cascade lifts — see v4.2 §3)
- §9 documents the full GRS computation **and** the `verdict: excluded` derived from the cascade trigger (verdict prevails for deployment per v4.2 §2.3)

### 0.3 Scope boundary

- **Chain scope:** Assess the strategy as deployed on its primary chain. Cross-chain variants (e.g., Curve LP on Ethereum vs. Arbitrum) require separate assessments if protocol deployments differ materially.
- **Maturity scope (Pendle):** Each maturity is a distinct L3 — PT-sUSDe-18JUN2026 and PT-sUSDe-7MAY2026 are two strategies.
- **Pool scope (LP):** Each pool pair is distinct — frxUSD/msUSD and frxUSD/crvUSD are two strategies even if both are Curve+Convex.

### 0.4 Data freshness

All live metrics (TVL, APY, price, pool depth) must be dated within 7 days of assessment. If staler, refresh before scoring.

---

## 1. ProtocolRisk Computation — Multi-Protocol Rule (§4.2.2)

**Formula:** `ProtocolRisk = max(Pi) + (N-1) × 0.20 × mean(Pi)`, where N = count of distinct protocols, capped at **N ≤ 3**.

### 1.1 Protocol enumeration rules

Count each protocol that contributes **material technical surface** to the strategy:

**Counted protocols** (examples):
- The AMM itself (Curve, Balancer, Uniswap V3/V4, Ekubo, Velodrome, Aerodrome)
- Booster / reward layer if separately audited (Convex, Aura, Beefy)
- The lending protocol if users borrow against the LP token
- The vault curator protocol (Morpho Blue, Euler, Yearn, Ipor)
- Token issuer protocols that are **NOT** already captured via L2 asset dual-assessment
- The underlying token issuer ONLY if the strategy depends on protocol-level flows distinct from the token's L2 scoring (rare; usually already captured via L2)

**NOT counted** (to avoid double-counting):
- L1 issuer protocols of assets ALREADY scored at L2 (the L1 score is captured in the L2 ARS via dual-assessment)
- Wrapper contracts that are thin passthroughs (e.g., plain ERC-4626 wrapper with no custom logic)
- Oracle providers (captured in S3 Oracle Risk for Type 1, or S3 Dual Contract Surface for Type 2A)

### 1.2 Worked examples

**Example A:** Curve + Convex stablecoin LP
- Curve PRS = 3.93, Convex PRS = 5.39
- N = 2, max = 5.39, mean = 4.66
- ProtocolRisk = 5.39 + (2-1) × 0.20 × 4.66 = 5.39 + 0.93 = **6.32**

**Example B:** Spark wstETH-WETH looping
- Spark PRS = 2.37 (only Spark; Lido is captured via wstETH L2)
- N = 1, ProtocolRisk = 2.37

**Example C:** Morpho MetaMorpho vault with Gauntlet curator deploying across Aave + Morpho Blue markets
- Morpho Vaults PRS = 2.55, Morpho Blue PRS = 1.87, Aave V3 PRS = 2.64
- N = 3 (at cap), max = 2.64, mean = 2.35
- ProtocolRisk = 2.64 + (3-1) × 0.20 × 2.35 = 2.64 + 0.94 = **3.58**

**Example D:** Pendle LP with underlying asset from a 4th protocol
- Pendle PRS = 4.24, underlying token issuer at L1 already captured via L2 → N = 1
- ProtocolRisk = 4.24

**Example E:** Strategy touching 4 distinct protocols (rare)
- N = 4 → **EXCEEDS CAP**
- ProtocolRisk = 9.0 (auto-assigned per §4.2.2 hard cap)
- Flag to CEO for compositional-complexity review

### 1.3 Output validation

- `ProtocolRisk = min(formula_result, 10.0)` — cap at 10
- Round to 2 decimals
- Document which protocols were enumerated and why

---

## 2. AssetRisk Computation — Multi-Asset Rule (§4.2.3)

### 2.1 Single-asset strategies

`AssetRisk = Ai` (the single asset's L2 ARS).

Applies to:
- Type 1 looping (collateral + borrow are the SAME L1 risk — e.g., wstETH-ETH loops; score the collateral)
- Type 3 single-collateral lending strategies (direct N=1 isolated markets, or curated vaults concentrated in one collateral type)
- Pendle PT that is pure-underlying exposure (not LP)

### 2.2 Paired-asset strategies (LP pools)

`AssetRisk = 0.75 × max(Ai) + 0.25 × mean(Ai)`

Applies to:
- All Type 2A LP/AMM (Curve, Ekubo, Velodrome, etc. paired pools)
- Type 2B Pendle LP where LP token is paired (e.g., PT+SY composite)

**Rationale:** The 0.75/0.25 weighting captures that LP positions accumulate risk disproportionately toward the riskier asset during depeg/stress scenarios — the lower-risk leg cannot compensate fully.

### 2.3 Multi-asset strategies (>2 assets, e.g., tri-pool)

`AssetRisk = 0.60 × max(Ai) + 0.30 × second-max(Ai) + 0.10 × mean(Ai)`

Applies to Curve tri-pools, balance-weighted LPs with 3+ constituents.

### 2.4 Basket-index strategies (e.g., ETH+ Reserve RToken)

Treat as **single-asset** using the RToken's L2 ARS directly. The basket composition is already captured in the L2 A4 scoring.

### 2.5 Worked examples

**Example A:** frxUSD/msUSD Curve LP
- frxUSD ARS = 4.55, msUSD ARS = 4.95
- AssetRisk = 0.75 × 4.95 + 0.25 × 4.75 = 3.71 + 1.19 = **4.90**

**Example B:** Pendle LP-tETH (single underlying)
- tETH ARS = 5.15
- AssetRisk = **5.15**

**Example C:** wstETH-WETH Spark loop
- wstETH ARS = 3.20 (collateral); WETH ARS = 1.0 (borrow)
- Treated as single-asset (collateral) per §2.1
- AssetRisk = **3.20**

**Example D:** ynETHx/WETH Curve LP (shows max dominance)
- ynETHx ARS = 6.60 EXCLUDED → **Strategy EXCLUDED via L2 cascade** per §0.2.2
- (Formula moot — not scored)

---

## 3. Type 1 — Looping / Leverage Strategies

**Applicable to:** Recursive borrow-redeposit positions on lending markets. Common examples: wstETH→ETH→re-deposit on Spark/Vesu/Aave; frxETH→ETH on Ipor; cbETH→ETH.

**Formula:** `StrategySpecificRisk = 0.35·S1 + 0.25·S2 + 0.20·S3 + 0.20·S4`

### 3.1 S1 — Leverage-Adjusted Correlation Risk (35%)

**Verify:**
- Compute historical correlation between collateral and borrowed asset (365d daily returns)
- Identify max historical depeg under stress (e.g., Jun 2022 Curve stETH depeg 4-7%, Apr 2024 ezETH 80%, Jan 2025 USD0++ 10.7%)
- Determine effective leverage ratio (how many loop iterations? effective LTV?)

**Data sources:** DefiLlama borrow positions, protocol's UI (show effective leverage), Dune dashboards, on-chain position reconstruction

**Edge case:** "Delta-neutral LST looping" (e.g., wstETH borrowing ETH back) appears low-risk but oracle-price divergence during stress can trigger cascading liquidations even when NAV is stable. Score at least 4 if leverage > 10x.

### 3.2 S2 — Stress Liquidation Buffer (25%)

**Verify:**
- Current position LTV (from protocol UI or on-chain)
- Liquidation threshold LTV (from protocol risk parameters)
- Apply historical max depeg scenario: does position survive?
- Buffer = (liquidation_LTV - current_LTV) / liquidation_LTV, adjusted for worst-case oracle deviation

**Data sources:** Protocol risk parameter pages (Aave, Spark, Morpho Blue market configs), Chaos Labs / LlamaRisk public dashboards

**Special case — Type B adaptive liquidation:** Some protocols (e.g., Morpho Blue IRM markets) have dynamic liquidation parameters. Use the WORST-CASE configured parameter, not current snapshot.

### 3.3 S3 — Oracle Risk (20%)

**Verify:**
- Primary oracle source: Chainlink, Pyth, RedStone, TWAP-only, or custom?
- Heartbeat (how often does oracle update?)
- Source count (aggregator reading how many underlying feeds?)
- Circuit breaker presence (max deviation cap per update?)
- Fallback logic (if primary fails, what happens?)

**Data sources:** Oracle contract addresses (data.chain.link), protocol docs risk section, Chaos Labs oracle dashboards

**Cascade rule:** If protocol uses EigenLayer-style operator attestation instead of cryptographic oracle, score ≥ 7.

### 3.4 S4 — Unwind Liquidity (20%)

**Verify:**
- Total deposit size (institutional-relevant: $1M, $5M, $10M scenarios)
- DEX routes available to unwind
- Slippage at each size (CoW Swap, 1inch, aggregator quotes)
- Bridge dependencies if multi-chain
- Time-to-full-exit under stressed conditions

**Critical check:** For Starknet-isolated strategies, include bridge latency/cost in unwind scenario. Starknet→Ethereum StarkGate bridge is 1-12hr typically.

### 3.5 Strategy-type-specific edge cases

- **Spark wstETH-WETH looping:** wstETH/ETH oracle uses Chainlink rate-providing contract with 24hr heartbeat — S3 typically 3-4 band
- **Ipor-hosted looping:** Adds Ipor protocol layer (separately audited) — counts toward Multi-Protocol N
- **Vesu (Starknet):** Vesu C3 governance still maturing — factor into ProtocolRisk, not S3

---

## 4. Type 2A — LP / AMM Classic Strategies

**Applicable to:** LP positions in Curve, Balancer, Uniswap V3/V4, Ekubo, Velodrome, Aerodrome. With or without booster (Convex, Aura, Beefy, Stake DAO).

**Formula:** `StrategySpecificRisk = 0.35·S1 + 0.30·S2 + 0.20·S3 + 0.15·S4`

### 4.1 S1 — IL Risk (35%)

**Verify:**
- Asset correlation (365d daily returns on CoinGecko/DefiLlama)
- Historical IL on the specific pool (if available via Dune, Convex pool analytics)
- Pool weight structure (50/50, 80/20, Stableswap concentrated, etc.)

**Curve Stableswap heuristic:** For stable-stable pairs (USDC/USDT, crvUSD/USDC), IL is structurally capped at <0.1% under pool's amplification factor — score 1-2.

**LST-LST heuristic:** For LST/LST pairs sharing same underlying (wstETH/cbETH both pegged to ETH), IL 0.1-0.5% — score 3-4.

### 4.2 S2 — Depeg Correlation (30%)

**Key question:** If one asset in the pair depegs, does the other likely follow?

**Scoring patterns:**
- Independent mechanisms (USDC vs frxUSD): score 1-2
- Same issuer family (frxUSD vs sfrxUSD — both Frax): score 4-5 partial dependency
- Same underlying (wstETH vs stETH): score 7-8 total dependency (atomic convertibility + same issuer)
- Same protocol cascade (rsETH vs ezETH — both EigenLayer LRTs subject to April 2024-style panic): score 7-8

### 4.3 S3 — Dual Contract Surface (20%)

Pool contract + booster contract attack surface. See §4.3.2 in risk_methodology.md for rubric.

**Curve + Convex heuristic:** Pool audited 3+ times tier 1, Convex audited 3+ times tier 1, both operational >4 years, zero incidents → score 1-2.

**Ekubo + Starknet isolation:** Ekubo audited but younger protocol, Starknet-specific implementation — score 4-5 even if mature on other chains.

### 4.4 S4 — Reward Token Risk (15%)

- Fees-only (no emissions): 1-2
- Major governance token (CRV, CVX, BAL, AERO) top-100 mcap: 3-4
- Mid-cap gov token: 5-6
- Small-cap / farm-and-dump: 7-8

### 4.5 Edge cases for Type 2A

- **Starknet Ekubo LPs:** Apply single-venue penalty in S4/S5 since Starknet bridge-out adds unwind complexity
- **Balancer weighted pool with LST constituent:** Factor constituent's L2 ARS into AssetRisk via §2.3 multi-asset rule
- **Deprecated pool** (e.g., no more active incentives, declining TVL): Flag as monitoring signal, consider elevating S3 if booster contract is being migrated
- **Concentrated-liquidity LP (Uniswap V3, V4, Bunni, Algebra, Trader Joe Liquidity Book):** The S1 IL rubric assumes constant-product or stableswap bonding curves; concentrated-LP positions go out-of-range and earn zero fees, with asymmetric IL accumulation as price moves through the band.
  - **Stable-stable concentrated LP** (e.g., USDC/USDT 0.01% V3 tight band): apply standard scoring; S1 typically 1–3.
  - **Volatile-stable or volatile-volatile concentrated LP** (e.g., ETH/USDC 0.05%): apply **S1 ≥ 6 floor** for range-management risk.
  - **Active-managed concentrated LP** (Arrakis, Gamma, Bunni LIT, Steer auto-rebalance): apply **S1 ≥ 4 floor** instead of ≥ 6 (the rebalancer reduces but does not eliminate the risk).
  - S4 Reward Token Risk typically 1–2 (fees-only, no emissions) unless paired with a third-party booster.

---

## 5. Type 2B — LP Pendle Strategies

**Applicable to:** Pendle AMM pools splitting yield-bearing tokens into PT (Principal Token) + SY (Standardized Yield) or PT + YT (Yield Token).

**Formula:** `StrategySpecificRisk = 0.35·S1 + 0.30·S2 + 0.20·S3 + 0.15·S4`

### 5.1 S1 — Yield Divergence Risk (35%)

**Verify:**
- Underlying yield source: ETH staking? Ethena funding? Treasuries? First-loss tranche?
- 365d yield volatility (annualized standard deviation of APY)
- Correlation with market regime (does yield collapse when BTC/ETH funding flips negative?)

**Heuristics:**
- ETH staking (3-4%, stable): 1-2
- LST-based lending rate (Aave stETH supply rate): 3-4
- Ethena sUSDe (3-60% range, funding-dependent): 7-8
- Junior tranche / first-loss (jrUSDe-style): 9-10 — highly unpredictable

### 5.2 S2 — Underlying Asset Risk (30%, mapped from L2)

**Mapping table (from methodology §4.3.3):**

| L2 ARS | S2 Score |
|--------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 5.0 | 4 |
| 5.0 – 6.5 | 6 |
| 6.5 – 7.5 | 8 |
| > 7.5 | 10 |

**Cascade rule:** If underlying L2 ARS > 6.5 (v3.7 L2 hard cap), the strategy `verdict` is forced to **EXCLUDED** via §0.2.2 regardless of composite GRS. Per v4.2, S2 (and all other components / criteria / composite GRS) is still computed and published — the verdict field carries the cascade exclusion; the GRS provides continuous measurement of how the strategy would score if the cascade lifted. Assets in WATCHLIST band (ARS 6.0–6.5) trigger §5 Rule 3 concentration cap, not exclusion.

### 5.3 S3 — Maturity Risk (20%) — CRITICAL

**Verify:**
- Exact maturity date (from Pendle app or contract)
- Days remaining to maturity (vs. assessment date)
- Pool TVL decay trend (check last 30 days — is TVL declining as maturity nears?)
- Yield exposure increases nonlinearly in final weeks

**Near-maturity escalation pattern:**
- Maturity > 6mo: comfortable — score 1-2
- Maturity 4-6mo: score 3-4
- Maturity 2-4mo: score 5-6
- Maturity < 2mo: score 7-8 (operational attention required)
- **Maturity < 2 weeks: score 9-10** — treat as near-expiry; strongly recommend exit or roll

**Worked example:** Pendle LP-tETH-30APR2026 assessed 2026-04-21 → **9 days to maturity → S3 = 9** → material pressure on GRS. Recommend position exit or roll to next maturity.

### 5.4 S4 — Pool Liquidity (15%)

- Pool TVL > $50M: 1-2
- Pool TVL $25-50M: 3-4
- Pool TVL $10-25M: 4-5
- Pool TVL $5-10M: 6
- Pool TVL < $5M: 7-8
- Pool essentially illiquid (<$500K): 9-10

**Special consideration:** Pendle pool depth decays as maturity approaches (arbitrageurs close positions). Score S4 against **current** TVL, not launch TVL.

### 5.5 Pendle-specific edge cases

- **PT-only position (no LP):** Treat as single-asset strategy; S1 Yield Divergence may score lower (you've locked yield); S3 Maturity still material
- **YT position:** Highly leveraged yield-only exposure; S1 typically 8-10 due to volatility
- **LP-Boosted position (with Pendle boost):** Additional protocol contract surface; factor into ProtocolRisk if Pendle Boost introduces distinct contract
- **Pendle V2 vs V3:** V3 has different AMM math (PT/SY only), V2 had PT/YT pools — document which version

---

## 5b. Type 2C — Pendle PT (Bond-like) Strategies

**Applicable to:** Holding standalone Pendle Principal Tokens (PT) without LP exposure. The user buys PT at a discount and either holds to maturity (redeem 1:1 for the underlying yield-bearing asset) or exits early via Pendle's PT/SY orderbook. Key distinction from Type 2B: **no IL, no AMM bonding-curve exposure** — the position is bond-like, dominated by the underlying asset's solvency through maturity.

**Formula:** `StrategySpecificRisk = 0.40·S1 + 0.30·S2 + 0.20·S3 + 0.10·S4`

### 5b.1 S1 — Underlying Asset Risk (40%)

This is the dominant factor for PT-only positions. PT's redemption ratio is fixed at maturity; the only way the position is materially impaired is if the underlying yield-bearing asset itself depegs, becomes illiquid, or has a backing failure between purchase and maturity.

**Mapping table** (same as Type 2B S2):

| L2 ARS | S1 Score |
|--------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 5.0 | 4 |
| 5.0 – 6.5 | 6 |
| 6.5 – 7.5 | 8 |
| > 7.5 | 10 |

**Cascade rule:** If underlying L2 ARS > 6.5 (v3.7 L2 hard cap), the strategy `verdict` is forced to **EXCLUDED** via §0.2.2 regardless of composite GRS. Per v4.2, S1 (and all other components / criteria / composite GRS) is still computed and published — the verdict field carries the cascade exclusion; the GRS provides continuous measurement of how the strategy would score if the cascade lifted. Assets in WATCHLIST band (ARS 6.0–6.5) trigger §5 Rule 3 concentration cap, not exclusion.

### 5b.2 S2 — Maturity Risk (30%)

Same rubric as Type 2B S3 — time to expiration drives operational attention as exit liquidity decays.

| Maturity | S2 |
|----------|-----|
| > 6 months | 1–2 |
| 4–6 months | 3–4 |
| 2–4 months | 5–6 |
| < 2 months | 7–8 |
| < 2 weeks | 9–10 (recommend exit/roll) |

### 5b.3 S3 — Yield Source Quality (20%)

Measures whether the embedded yield mechanism can break **before maturity**, impairing PT redemption. This factor is what distinguishes PT-on-sUSDe (S3 7–8: funding-rate dependent, but redemption survives unless backing impaired) from PT-on-USDtb (S3 1–2: T-bill backing is structurally durable).

| Score | Criteria |
|-------|----------|
| 1–2 | RWA T-bill backing, ETH staking — embedded yield extremely stable; near-zero failure risk pre-maturity |
| 3–4 | Lending rate (Aave/Morpho/Spark supply rate) — rate-dependent but mechanism cannot fail |
| 5–6 | Hybrid mechanism (e.g., LST-based lending) or moderate yield-source variability |
| 7–8 | Funding-rate-dependent (Ethena sUSDe, BasisOS) — yield can collapse but PT redemption survives unless backing impaired |
| 9–10 | First-loss tranche, junior position, or experimental/novel yield source — material risk that the underlying is impaired before maturity, breaking PT redemption |

**Verify:**
- What backs the underlying yield? (T-bills, staking, lending, funding-rate, options-selling, first-loss tranche)
- Has the underlying ever lost peg or had a backing impairment event?
- Is the yield mechanism dependent on a specific market regime (e.g., positive funding) that could invert?

### 5b.4 S4 — PT Exit Liquidity (10%)

Pendle PT/SY orderbook depth at the relevant maturity. Used only if early exit is required; for hold-to-maturity strategies, S4 is moderated.

| Score | Criteria |
|-------|----------|
| 1–2 | PT/SY pool TVL > $50M; $1M PT exit < 0.5% slippage |
| 3–4 | PT/SY pool TVL $25–50M; $1M exit 0.5–1% slippage |
| 4–5 | PT/SY pool TVL $10–25M; $1M exit 1–2% slippage |
| 6 | PT/SY pool TVL $5–10M; $1M exit 2–3% slippage |
| 7–8 | PT/SY pool TVL < $5M |
| 9–10 | No active PT/SY orderbook (must hold to maturity), or MISSING |

**Hold-to-maturity moderator:** If the deployment plan explicitly intends to hold PT to maturity with no early-exit reliance, S4 may be moderated by **1 band downward** (e.g., 7→6, 5→4). Document the hold-to-maturity intent and the matching maturity timeline in the assessment.

### 5b.5 Type 2C edge cases

- **PT redemption mechanics:** PT can always be redeemed 1:1 for the underlying SY at maturity, regardless of orderbook liquidity. S4 only matters for pre-maturity exit.
- **Yield-source vs. asset-issuer split:** The underlying L2 asset's ARS captures issuer-level risk (e.g., Ethena protocol risk for sUSDe). S3 separately captures the **mechanism's** survival probability, which can diverge from the issuer's solvency in stress (e.g., Ethena issuer healthy but funding regime inverts → sUSDe yield collapses but redemption ratio holds).
- **Multi-protocol enumeration:** Pendle is the only counted protocol for pure PT holding (N=1). The underlying asset's issuer is captured via L2 dual-assessment per §1.1; do not double-count.
- **Position sizing (Section 5 Rule 4):** PT positions are subject to a 20% per-maturity cap, 20% of total PT supply at that maturity, and a 30% aggregate cap across all maturities of the same underlying.

### 5b.6 Worked example

**Strategy:** Hold PT-sUSDe-25SEP2026 to maturity (assessed 2026-04-25, 5 months to maturity)

- **L1 cascade:** Pendle PRS 4.24 → APPROVED
- **L2 cascade:** sUSDe ARS 4.85 → APPROVED (< 6.0 cap)
- **ProtocolRisk:** N=1 (Pendle only; Ethena captured via L2) → **4.24**
- **AssetRisk** (single-asset): sUSDe ARS = **4.85**
- **StrategySpecificRisk:**
  - S1 Underlying Asset Risk = 4 (sUSDe ARS 4.85 maps to band 4)
  - S2 Maturity Risk = 4 (5 months out)
  - S3 Yield Source Quality = 7 (Ethena funding-rate dependent)
  - S4 PT Exit Liquidity = 3 (Pendle PT-sUSDe orderbook ~$200M+); moderated to 2 if hold-to-maturity confirmed
  - StrategySpecificRisk = 0.40×4 + 0.30×4 + 0.20×7 + 0.10×2 = 1.60 + 1.20 + 1.40 + 0.20 = **4.40**
- **GRS** = 0.35×4.24 + 0.25×4.85 + 0.40×4.40 = 1.484 + 1.213 + 1.760 = **4.46** → **APPROVED**

---

## 6. Type 3 — Lending Strategies

**Applicable to:** Depositing assets into a lending position with 1 or more underlying markets:
- **N=1 (direct lending):** single isolated lending market — Morpho Blue isolated market, Felix isolated market, single Euler vault. No curator. Parameters fixed at market creation (immutable for Morpho Blue and similar).
- **N≥2 (curated vault):** curated lending vault that allocates across multiple markets — MetaMorpho, Euler curated, Yearn V3, Ipor.

**Formula:** `StrategySpecificRisk = 0.30·S1 + 0.25·S2 + 0.20·S3 + 0.15·S4 + 0.10·S5`

### 6.1 S1 — Risk Management Quality (30%)

S1 has two evaluation modes — pick by N. The scoring weight (30%) and band semantics are identical; only the rubric content differs.

#### Mode A — Direct single-market (N=1)

For a direct deposit into a single isolated market, the "risk management" is the static parameter set chosen at market creation. For Morpho Blue and similar, these parameters are immutable post-creation, so the score is permanent unless the entire market is migrated.

**Verify:**
- **LLTV** vs. the collateral's L2 ARS — does the LLTV leave adequate buffer for the collateral's stress profile?
- **Oracle**: tier (Chainlink, Pyth, RedStone vs. custom), source count, heartbeat, circuit breaker presence, fallback logic
- **IRM** (interest rate model): battle-tested (e.g., Morpho AdaptiveCurveIRM) vs. novel?
- **Market creator**: doxxed reputable team coordinating with collateral issuer? Pseudonymous? Anonymous?
- **Coordination**: was the market created with the collateral issuer's involvement (canonical market) or by an arbitrary third party?

**Heuristic:**
- LLTV ≥10pp below comparable mainstream markets for the collateral risk band, Tier-1 oracle with multi-source + circuit breaker, AdaptiveCurveIRM (or equiv.), creator is doxxed: **score 1–2**
- LLTV with 5–10pp buffer, Tier-1 oracle (single-source ok), battle-tested IRM, creator known: **score 3–4**
- LLTV typical-but-tight, mixed-tier oracle, known IRM, creator semi-known: **score 5–6**
- LLTV aggressive vs. collateral ARS, untested oracle or novel IRM, anonymous creator: **score 7–8**
- LLTV inappropriate for collateral risk, unverified oracle, untested IRM, or unverifiable creator: **score 9–10**

#### Mode B — Curated vault (N≥2)

**Verify:**
- Who is the curator? (Gauntlet, Steakhouse, Re7, B.Protocol, MEV Capital, Block Analitica, etc.)
- Track record length
- Public risk reports / decision logs
- Audit coverage of curator processes

**Tier 1 curators** (Gauntlet, Steakhouse, Re7, B.Protocol, MEV Capital): score 1-4
**Tier 2 curators** (emerging but known teams, with risk reports): score 4-5
**Semi-known / limited public process:** score 6
**Unknown curator:** score 7-8

### 6.2 S2 — Collateral Composition (25%)

Compute **weighted average L2 ARS of vault's allowed collateral assets** (weighted by current allocation).

**Mapping:**

| Weighted Avg L2 ARS | S2 Score |
|--------------------|----------|
| ≤ 3.0 | 2 |
| 3.0 – 4.5 | 4 |
| 4.5 – 6.0 | 6 |
| > 6.0 | 8 |
| Unknown collateral | 9 |

**Cascade rule:** If ANY vault collateral has `eligibility == "excluded"` at L2, **strategy is EXCLUDED** even if the vault holds only a small allocation of that collateral. The user cannot selectively avoid bad markets once deposited.

### 6.3 S3 — Utilization Spike Risk (20%)

**Verify:**
- Historical utilization rate (90d, 180d) — Aave/Morpho dashboards + Dune
- Count episodes where utilization exceeded 90%
- Average duration of such episodes
- Whether user impact was observed (locked redemptions, bad debt)

### 6.4 S4 — Collateral Correlation Risk (15%)

Different from S2 (composition): this measures the **diversification** of the vault's collateral exposure.

- Vault with 3+ uncorrelated collateral classes (e.g., blue-chip USD + LSTs + BTC): 1-2
- 2 dominant classes, moderate correlation: 4-5
- Single dominant collateral: 7-8

### 6.5 S5 — Bad Debt History (10%)

- Zero bad debt, vault >6mo: 1-2
- Zero bad debt, vault 3-6mo: 3-4
- Bad debt <0.01% TVL, resolved quickly: 4-5
- Bad debt >0.01% or unresolved: 7-8
- Vault <3mo with no history: 9-10

### 6.6 Lending edge cases

- **Direct single-market lending (N=1):** Score S1 under Mode A (market configuration quality). S4 (Collateral Correlation) typically lands **7–8 by structure** — single-collateral exposure is concentration risk by definition, and the lender cannot rebalance once deposited. Lender-side S2 maps from the single collateral's L2 ARS via the standard table; no weighted average needed.
- **MetaMorpho vault with market-level concentration:** If a single underlying market holds >50% of vault TVL and that market's collateral has `ARS > 5.0`, elevate S4 to 7+ — the curator's diversification benefit collapses.
- **Euler v2 vaults:** E-mode allocations may have tighter risk parameters — verify per-vault configuration, not protocol-wide.
- **Yearn V3 strategies with multiple allocators:** Each underlying strategy is its own lending venue — treat vault as multi-market (N≥2) and score S2 on aggregate collateral, S1 on the curator/manager process.
- **Borrower-side leverage on a single market:** This is **Type 1 (Looping)**, not Type 3. The lender side of the same market is Type 3 (N=1).

---

## 7. Cross-Cutting Considerations

### 7.1 Near-term maturity exit signal (Pendle-specific but generalizes)

Any strategy with <30 days to maturity/expiration/key event warrants:
- S3 (or equivalent maturity/time factor) at band 7-10
- CEO notification before new deployments
- Explicit documentation in `material_events` of expected exit timeline

### 7.2 Multi-chain cascade

For strategies deployed on secondary chains (Starknet, Base, Arbitrum):
- Primary assessment on canonical chain
- Chain-specific adjustments to S4 Unwind Liquidity (include bridge costs + latency)
- Register as separate assessment file if protocol deployment differs (e.g., Spark on Ethereum vs Spark on Gnosis)

### 7.3 Foregone-conclusion exclusion (documentation-only L3)

When L1 or L2 cascade has already determined exclusion (§0.2), produce abbreviated L3 assessment:

**Abbreviated format:**
```
# L3 Strategy — <name>
VERDICT: EXCLUDED via cascade
CASCADE PATH: <L1 protocol excluded OR L2 asset excluded>
TRIGGER RULE: <methodology §>
RECOMMENDED ACTION: <0% allocation; wind-down within 7 days if active>
```

Skip S-factor scoring entirely. Document cascade path and move on.

### 7.4 Wind-down urgency matrix

When an active strategy becomes EXCLUDED via re-scoring:

| Current exposure | Cascade severity | Wind-down timeline |
|-----------------|------------------|--------------------|
| < 5% of vault | Any | 14 days |
| 5-15% of vault | L2 A2 ≥ 9 OR L1 C3 ≥ 8 | **7 days** |
| 5-15% of vault | ARS > 6.5 (not A2) | 14 days |
| > 15% of vault | Any | **7 days** |
| > 30% of vault | Any | **Immediate** (coordinate with CEO) |

---

## 8. GRS Computation and Eligibility

### 8.1 Final formula

```
GRS = 0.35 × ProtocolRisk + 0.25 × AssetRisk + 0.40 × StrategySpecificRisk
```

Round to 2 decimals.

### 8.2 Eligibility thresholds

| GRS Range | Status |
|-----------|--------|
| ≤ 7.5 | **APPROVED** — eligible for vault deployment, subject to Section 5 Rules |
| > 7.5 | **EXCLUDED** |

### 8.3 Section 5 position sizing caps (reference)

After GRS eligibility confirmed, apply Section 5 Rules 1-4 for vault allocation ceilings:
- **Rule 1:** Vault Risk Score cap (aggregated)
- **Rule 2:** Protocol concentration cap (based on L1 PRS)
- **Rule 3:** Asset concentration cap (based on L2 ARS)
- **Rule 4:** Position sizing cap per strategy type

The binding cap is `min(Rule 2, Rule 3, Rule 4)`. Document this in the output.

### 8.4 Missing data protocol

Per methodology §4.2.5, when data for a specific S-factor is unverifiable:
- Score that factor at **band 9-10** (conservative MISSING)
- If MISSING pushes GRS > 7.5 → strategy EXCLUDED pending data recovery
- Document gap in Limitations; commit to re-score once data available

**Do NOT** use band midpoint or optimistic assumption for MISSING data — default to conservative exclusion.

---

## 9. Output Contract — What a Complete L3 Assessment Must Contain

### 9.1 Required sections

1. **Header:** Asset/strategy name, assessment date, methodology version, analyst, status
2. **Summary Table:** Strategy type, ProtocolRisk, AssetRisk, StrategySpecificRisk, GRS, verdict
3. **§0 Pre-scoring:** classification decision, L1/L2 cascade validation, eligibility gate result
4. **§1 ProtocolRisk:** protocol enumeration, Multi-Protocol Rule calculation
5. **§2 AssetRisk:** asset enumeration, Multi-Asset Rule calculation
6. **§3 StrategySpecificRisk:** per-S-factor scoring with rationale (2-3 sentences each + data source)
7. **§4 GRS composite:** final computation with component breakdown
8. **§5 Eligibility check:** hard caps + threshold check
9. **§6 Section 5 position sizing:** binding cap determination
10. **§7 Material events / monitoring:** near-term maturity, wind-down triggers, re-review signals
11. **§8 Strengths and risks**
12. **§9 Sources:** documented URLs and on-chain verification methods
13. **§10 Limitations:** flagged gaps in data
14. **Footer:** `ASSESSMENT_STATUS: COMPLETE`, `PUBLISHER_TRIGGER: L3|<name>|<file_path>|<registry_path>`, `NOTION_SYNC: <url>` (after sync)

### 9.2 Strategy Registry entry format

After assessment, add entry to `registries/strategy_registry.json` (create if missing) with:
- strategy_name, vault (fyUSDC/fyETH/fyWBTC), strategy_type (1/2A/2B/3)
- protocols_list, assets_list
- ProtocolRisk, AssetRisk, StrategySpecificRisk, GRS
- S-factors dict with scores and rationale
- eligibility, hard_cap_triggered, binding_concentration_cap
- notion_url, assessment_file, material_events, monitoring_signals

### 9.3 Abbreviated output for foregone-conclusion exclusions

See §7.3. Truncated format acceptable; must still document cascade path for audit trail.

---

## 10. Worked Example — Reference Assessment

**Strategy:** `reUSD/sfrxUSD Curve+Convex LP` (fyUSDC)

### §0 Pre-scoring

- **Type:** 2A (LP / AMM Classic with booster)
- **Protocols:** Curve (PRS 3.93), Convex (PRS 5.39)
- **Assets:** reUSD (ARS 4.55), sfrxUSD (ARS 4.55)
- **L1 cascade:** All APPROVED
- **L2 cascade:** All APPROVED
- **Eligibility gate:** PASS — proceed to scoring

### §1 ProtocolRisk (Multi-Protocol Rule)

- N = 2, max = 5.39, mean = 4.66
- ProtocolRisk = 5.39 + (2-1) × 0.20 × 4.66 = 5.39 + 0.932 = **6.32**

### §2 AssetRisk (Multi-Asset Rule)

- reUSD = 4.55, sfrxUSD = 4.55, max = 4.55, mean = 4.55
- AssetRisk = 0.75 × 4.55 + 0.25 × 4.55 = **4.55**

### §3 Strategy-Specific Risk (Type 2A)

- S1 IL Risk = 3 (correlated stables, Curve Stableswap)
- S2 Depeg Correlation = 4 (different issuers Resupply vs Frax)
- S3 Dual Contract Surface = 2 (Curve + Convex both mature, multi-audit)
- S4 Reward Token Risk = 3 (CRV + CVX top-100)
- StrategySpecificRisk = 0.35·3 + 0.30·4 + 0.20·2 + 0.15·3 = 1.05 + 1.20 + 0.40 + 0.45 = **3.10**

### §4 GRS

- GRS = 0.35 × 6.32 + 0.25 × 4.55 + 0.40 × 3.10
- GRS = 2.212 + 1.138 + 1.240 = **4.59**

### §5 Eligibility

- GRS 4.59 ≤ 7.5 → **APPROVED**
- No hard cap triggered

### §6 Section 5 binding cap

- Rule 2 (L1 Convex 5.39): 30% per vault
- Rule 3 (L2 both assets 4.55): 60% per vault
- Rule 4 (Type 2A): 25% per strategy
- **Binding: 25%** (Rule 4)

---

## 11. L3 Methodology Change Log

- **2026-04-21:** v3.4 initial field guide written from scratch. Operationalizes risk_methodology.md §4 with verification procedures, worked examples, edge cases encountered in Phase 1 L2 gap batches, and pre-scoring cascade gates.

---

**END — Layer 3 Strategy Assessment Field Guide v3.4**
