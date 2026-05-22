# Layer 2 — Asset Risk Assessment Methodology

**Current methodology version:** v4.1 (May 2026)
**Last L2-specific amendment:** v3.10 (A4 RWA Backing Class Taxonomy T/C/I, 2026-05-08)
**L2 rubric base:** unchanged since v3.4. Subsequent L2-specific changes: v3.7 (A3 Async Exit Credit + WATCHLIST band 6.0–6.5 + ARS hard cap raised from 6.0 to 6.5) and v3.10 (A4 Class T/C/I taxonomy). §0 cascade gate inherits v3.5/v3.6 L1 hard caps.
**Date:** Last meaningful edit 2026-05-08 (v3.10); thresholds revalidated against v4.1.
**Companion to:** `full-framework.md` (canonical source for rubrics, formulas, and hard caps). See `amendments/` for v3.7, v3.10 full text.

**Purpose:** Practical guide for scoring Layer 2 (asset-level) risk. Contains data sources, verification steps, and application notes for each criterion.

---

## 0. Pre-scoring Decisions (required before any criterion scoring)

### 0.1 What is — and is NOT — a Layer 2 asset

**L2 asset:** a **token whose risk is primarily about its own peg / backing / liquidity**, not about how it's used.

Examples of valid L2 assets:
- Fiat-backed stablecoins (USDC, USDT, USDP, USDG)
- CDP stablecoins (DAI, USDS, LUSD, BOLD, crvUSD, mkUSD, pmUSD, USR)
- Yield-bearing stablecoins (sUSDe, sUSDS, sfrxUSD, syrupUSDC)
- Synthetic dollars (USDe, deUSD, USDf, lvlUSD)
- **Tranche tokens** — junior/senior tranches of other assets (jrUSDe, reUSDe, USD0++) — see §7
- LSTs (stETH, wstETH, rETH, frxETH, ETHx, cbETH)
- LRTs (weETH, rsETH, ezETH, pufETH, rswETH)
- Wrapped assets (WETH, WBTC, tBTC, cbBTC)
- Native assets (ETH, BTC) — scored with defaults (§6)
- RWA tokens (OUSG, USDY, IONau) — apply RWA override in A4 (§5)

**NOT L2 assets** (these are L3 strategies, not underlyings):
- Vault receipt tokens of lending protocols: eUSDC-95 (Euler), gteUSDc (Morpho), aUSDC (Aave), yvUSDC (Yearn) → **L3 Type 3 Lending Vault**
- Pendle PT/YT/LP tokens: PT-apyUSD-18JUN2026, LP-jrUSDe-25JUN2026 → **L3 Type 2B LP Pendle**
- Looping position tokens: spark-wstETH-WETH-looping, fusnstETH (IPOR wrapper) → **L3 Type 1 Looping**
- AMM LP tokens: uniETH/frxETH Curve Convex LP, Ekubo WBTC/tBTC LP → **L3 Type 2A LP/AMM Classic**

When in doubt, ask: *"Does this token's risk come from what it represents (peg/backing), or from what it does (strategy mechanism)?"* If the latter, it's L3.

### 0.2 Dual-assessment requirement

Per `risk_methodology.md §3.0`, these asset types require a **separate L1 assessment of their issuing protocol**:

| Asset type | Example | L1 required |
|---|---|---|
| Yield-bearing stablecoins | sUSDe, pmUSD, sUSDS, USR | ✅ Yes |
| RWA-backed tokens | OUSG, USDY, IONau-backed, reUSD | ✅ Yes |
| Protocol-native tokens used as collateral | LQTY (if used), CRV (if used), ENA (if used) | ✅ Yes |
| CDP stablecoins (issued by a protocol) | DAI, USDS, LUSD, BOLD, crvUSD | ✅ Yes — protocol must be L1-scored |
| LSTs / LRTs | stETH, rETH, weETH, rsETH | ✅ Yes — staking issuer must be L1-scored |
| Tranche tokens | jrUSDe, reUSDe, USD0++ | ✅ Yes — tranche issuer AND underlying issuer (both L1s) |
| Pure fiat-backed stables | USDC, USDT, USDP, USDG, PYUSD | ✅ Yes — issuer (Circle, Tether, Paxos) L1-scored |
| Wrapped assets on Ethereum-native deposit contract | WETH, cbETH | ❌ No (no separate issuer protocol) |
| Decentralized BTC wrappers | tBTC, renBTC | ✅ Yes — Threshold Network, RenVM must be L1-scored |
| Custodial BTC wrappers | WBTC, cbBTC | ✅ Yes — BitGo, Coinbase must be L1-scored |
| Native assets | ETH, BTC | ❌ No |

**Handling:** L1 captures issuer-level risk (audits, governance, team); L2 captures asset-level risk (peg, liquidity, backing). They are computed independently and both carry forward into L3 strategy scoring.

**If L1 is excluded (hard cap triggered),** the L2 assessment is still produced for documentation, but any L3 strategy using the asset will inherit the L1 exclusion. Record this explicitly in the L2 "Notes / Cross-layer" section.

### 0.3 Scope — chain, wrapping, and cross-chain

- **Primary chain:** Assess the **Ethereum-native deployment** unless the asset is canonically issued on another chain (e.g., tBTC on Ethereum, GMX's GMX token on Arbitrum).
- **Bridged instances:** Bridged versions (e.g., USDC on Arbitrum via CCTP, WBTC on Polygon via PoS bridge) are **separate L2 assets** when the bridge mechanism materially changes backing trust. Document with an explicit "bridged variant" note.
- **Native multi-chain deployments:** When issuer mints natively on multiple chains with identical backing (e.g., Circle mints USDC natively on Solana, Base, Ethereum via CCTP) — assess the **aggregate** at the issuer level; note per-chain deployment differences in A3 (liquidity) and A5 (supply concentration).

### 0.4 Pre-scoring checklist

Before scoring any criterion, complete:

- [ ] Asset classified as L2 (not L3) per §0.1
- [ ] Dual-assessment requirement identified per §0.2 (L1 issuer recorded)
- [ ] Scope confirmed (chain, wrapping, multi-chain handling) per §0.3
- [ ] Contract address(es) verified on Etherscan
- [ ] Asset type category identified (stablecoin / LST / LRT / BTC wrap / RWA / tranche / native)
- [ ] Peg target stated (USD $1.00 / ETH 1:1 / BTC 1:1 / gold XAU 1oz / none-native)

---

## A1 — PEG MECHANISM (Weight: 20%)

**Objective:** Evaluate the type, robustness, and transparency of the asset's peg or value-maintenance mechanism.

**Data source priority (1 > 2 > 3):**
1. Protocol documentation — whitepaper, mechanism design docs, peg mechanism section
2. Smart contract source (Etherscan) — mint/redeem logic, oracle dependencies
3. Independent analyses — LlamaRisk, Blockworks Research, Steakhouse Financial, Block Analitica, academic papers
4. Audit reports (often document peg mechanism explicitly)
5. Cross-references from peer assets (e.g., comparing USDe mechanism description to sUSDe's)

**Verification sequence:**
1. **Identify peg type:** Fiat-backed / CDP / algorithmic / hybrid / native
2. **Map backing chain:** What backs the token? (cash, T-bills, crypto collateral, insurance treaties, gold, delta-neutral positions)
3. **Identify redemption path:** How does a holder get out at par? (direct with issuer / atomic on-chain / DEX-only / quarterly window / no redemption)
4. **Check attestation:** Is there a Proof-of-Reserves feed? Third-party audit?
5. **Novelty assessment:** How long has the mechanism been live in production?
6. **Stress-test understanding:** What breaks the peg? (custodian failure, collateral liquidation cascade, oracle manipulation, mass redemption)

**Critical checks:**
- **Redemption mechanism** is the single most important factor. A token with no redemption path relies entirely on market arbitrage → band 5-6 minimum.
- **Delay to redeem:** Instant on-chain (band 1-4) / T+1 (band 3-5) / Weekly (band 5-6) / Monthly+ (band 7-8) / Quarterly gating (band 7-9)
- **Novelty penalty:** Mechanisms <1 year in production, or that survived a single market cycle without a full stress test, score at minimum band 5-6.
- **Algorithmic peg (no collateral):** Automatic band 9-10 regardless of stability history (Terra/UST precedent).

**Edge cases:**
- **Tranche tokens** (jrUSDe, reUSDe, USD0++) — see §7 for scoring specifics. Base score inherits from underlying asset's A1 with a tranche mechanism penalty.
- **Wrapped BTC** (WBTC, tBTC, cbBTC) — classify by custody: custodial (BitGo for WBTC, Coinbase for cbBTC) vs. decentralized threshold-custody (tBTC). Custodial = band 3-5; decentralized = band 3-6 depending on signer set quality.
- **Wrapped ETH (WETH):** Smart-contract-guaranteed 1:1 wrap of native ETH. Band 1.
- **LSTs** (wstETH, rETH, frxETH): Exchange-rate appreciation (not $1 peg). Score peg mechanism on the **underlying staked ETH rate** + **validator slashing pass-through** design. Lido/Rocket Pool = band 3-4; newer = band 5-6.
- **LRTs** (weETH, rsETH): LST + restaking. Add EigenLayer-dependence in A4, but A1 should reflect the peg-to-ETH mechanism (similar to LSTs). Band typically 4-6.
- **Gold-backed** (PAXG, XAUT, IONau-backed pmUSD): Classify by physical allocation. Allocated bullion in regulated custody (PAXG, XAUT) = band 3-5; unallocated/in-situ (pmUSD via IONau) = band 7-9.

**What to look for:**
- Simple, well-understood, multi-year-tested mechanism = best (band 1-2)
- Novel design with clear mechanism but short track record = moderate (band 5-6)
- Opaque custody or algorithmic without collateral = red flag (band 9-10)

---

## A2 — DEPEG HISTORY (Weight: 25%)

**Objective:** Quantify historical deviations from peg — max deviation, duration, frequency.

**Data source priority:**
1. **CoinGecko / CoinMarketCap historical chart** (daily granularity primary)
2. **Dune Analytics dashboards** (for peg-specific tracking, e.g., USDC peg dashboard, stETH/ETH ratio dashboard)
3. **DEX-specific monitors:** Pharos Watch, Stablecoin Watch, Curve pool ratios, Uniswap V3 mid-price
4. **Rekt.news / DeFiLlama news** for depeg events linked to broader protocol incidents

**Verification sequence:**
1. **Pull full price history** since asset launch (not just the last 30d)
2. **Identify every deviation > 0.5% from peg target** sustained for > 1 hour
3. **For each event:** record (a) max % deviation, (b) duration, (c) date, (d) trigger/cause
4. **Classify events:** distinguish (a) **transient DEX dislocation** (arb-corrected within hours, no protocol issue) from (b) **mechanism depeg** (backing issue, redemption failure, cascade)
5. **Frequency count:** events per year
6. **Consolidate to rubric band:** use worst of (max deviation | duration | frequency) per rubric

**Critical checks:**
- **A2 ≥ 9 is a HARD CAP exclusion** (risk_methodology.md §4.2.1). Any asset with sustained peg loss >15% OR > 7 days duration is auto-excluded at L3.
- **Distinguish mechanism depeg vs. DEX noise.** A $1.02 print on a thin-liquidity DEX for 30 minutes is not a depeg. A $0.87 print across all venues for 48h is.
- **Launch-day prints** on extremely thin liquidity (first 24-72h of trading) should be **noted but not scored as mechanism depegs** unless cross-referenced with backing issues.
- **Yield-accrual tokens** (sUSDe, sUSDS, stETH rebasing) — peg is NOT $1.00 but an appreciating rate. A2 is measured against the **stated NAV / exchange rate**, not a fixed $1.

**Edge cases:**
- **NAV-accrual tokens that trade on open DEX without redemption floor** (e.g., reUSD, pmUSD): the DEX price can deviate from NAV freely. Score A2 on deviation vs NAV (per issuer's published rate), NOT vs $1. Document this choice explicitly.
- **USDC March 2023 SVB event**: $0.8774 ATL, ~72h recovery. This is band 5-6 under the rubric (5-15% deviation, 1-7 day duration). Benchmark event.
- **UST May 2022**: sustained loss >7d → band 9-10. Historical reference.
- **stETH June 2022**: 4-7% discount on Curve pools for several days during 3AC unwind. Band 5-6 (not 7-8 because withdrawals weren't live yet and the mechanism wasn't broken — just redemption-gated).
- **Pharos Watch count**: some assets accumulate many small depeg events (pmUSD has 419 per Pharos). Score by worst event, not count alone — but document the frequency as a risk signal.
- **Pegged LST/LRT** (assets targeting 1:1 to underlying): distinct from "1:1 to ETH" assets. E.g., ezETH targeted ETH-equivalent, depegged 80% in April 2024. Score on the specific peg target.

**What to look for:**
- Never depegged >0.5% sustained = band 1-2 (best)
- Minor discounts <2% during market stress, recovered fast = band 3-4
- Historical depeg 5-15% with multi-day duration = band 7-8
- Sustained loss >15% or >7d = band 9-10 hard cap

---

## A3 — LIQUIDITY DEPTH (Weight: 25%)

**Objective:** Assess ability to execute institutional-size trades without material slippage.

**Data source priority:**
1. **1inch / Paraswap / CowSwap aggregator quotes** — live $1M and $10M swap queries
2. **GeckoTerminal** (DEX pool depth + volume aggregation)
3. **CoinGecko 24h volume** (exchange-split, filtered to non-wash)
4. **CEX order book depth** (Binance, Coinbase, Kraken — use for assets with meaningful CEX presence)
5. **Curve / Balancer pool direct reads** for stablecoin depth
6. **Uniswap V3 subgraph** for concentrated-liquidity depth

**Verification sequence:**
1. **Simulate $10M swap** asset ↔ primary pair (USDC/WETH/WBTC) via aggregator
2. **Simulate $5M and $1M** swaps to build slippage curve
3. **Record 24h volume** (CEX + DEX, de-duplicated)
4. **Count major venues** with >$1M pool depth
5. **Verify CEX listings** (tier-1: Binance, Coinbase, Kraken, OKX; tier-2: Bybit, Gate, KuCoin)
6. **Scope per-chain liquidity** when asset exists on multiple chains — score based on primary deployment + note cross-chain depth

**Critical checks:**
- **$10M swap < 0.1% slippage** is band 1-2 (USDC, USDT territory)
- **$1M swap > 2% slippage** = band 9-10
- **Single-venue dependency** → worst-case band 7-10 regardless of depth on that venue
- **Curve pool ratios**: if asset has a Curve pool paired with its peg target (e.g., USDC/DAI, stETH/ETH), check current ratio. Pool imbalance >60/40 signals buying/selling pressure.

**Edge cases:**
- **DEX-only assets** (no CEX listing): cap at band 3-4 best-case unless DEX aggregate depth >$200M and 3+ independent pools.
- **Pendle PT/YT maturity proximity**: liquidity concentrates near maturity then collapses post-expiry. Not an L2 concern (Pendle tokens are L3 strategies) but may affect pool-level L2 token liquidity if pool includes PT/YT.
- **Cross-chain liquidity**: tBTC on Starknet (Ekubo ~$244K in our case) is shallow vs. tBTC on Ethereum. Score L2 for the asset globally, but flag per-chain deployment depth as a strategy-level consideration.
- **Bridged assets**: score the Ethereum-native deployment unless the non-Ethereum deployment is primary (e.g., tBTC on Ethereum is canonical; tBTC on Arbitrum is the bridged variant → separate consideration).
- **LP-paired-with-self** liquidity artifacts: an asset's "liquidity" may include large pools where it's paired against its own staked/wrapped version (e.g., stETH/wstETH pool). This is peg-arb liquidity, not exit liquidity. Discount accordingly.

**What to look for:**
- $10M swap + 24h volume >$500M + multi-venue = band 1-2
- $1M swap < 0.5% slippage + 2+ venues + $20M volume = band 5-6
- $1M swap > 2% slippage or single venue = band 9-10

---

## A4 — COLLATERAL BACKING (Weight: 20%)

**Objective:** Assess reserve composition, attestation cadence, custodian quality, legal enforceability.

**Data source priority:**
1. **Official Proof-of-Reserves dashboards** (e.g., paxos.com/usdg-transparency, circle.com/transparency)
2. **Third-party attestation reports** (Deloitte, KPMG, Withum, Grant Thornton, BDO)
3. **On-chain PoR feeds** (Chainlink Proof-of-Reserves for gold/BTC/cash)
4. **Arkham / Nansen on-chain analytics** for custody wallet tracking
5. **Issuer quarterly reports, 10-Ks (for public issuers like Paxos, Coinbase, I-ON Digital)**
6. **Etherscan token balance** of CDP vaults / Token Blender for crypto-collateralized stables

**Verification sequence:**
1. **Identify backing composition:** what % cash / T-bills / crypto / RWA / algorithmic / other?
2. **Locate attestation** — who, how recent, what scope?
3. **Verify custodian** — regulated entity? Jurisdiction? Insurance?
4. **Check for under-collateralization signals:** negative reserves, delayed attestation, missed reports
5. **Assess legal enforceability** (especially for RWA) — is there a lawyer-opinion letter? Jurisdictional enforceability?
6. **Look for live PoR** vs. point-in-time attestation (live = band 1-2; monthly = band 3-4; quarterly = band 5-6)

**Critical checks:**
- **Static reserves rule:** if backing is held in **static reserves** (cash, T-bills, ETH in smart contract) → score **A4 only at L2**. Don't double-penalize via L1.
- **Active operations rule:** if backing mechanism **IS the issuer's active operation** (e.g., Ethena delta-neutral hedging, Resolv's insurance RLP, RAAC's mining reserves) → A4 score **+ flag cross-dependency with L1 C5 and C4**. Apply the **more conservative** of the two resulting risk signals; do NOT double-penalize.
- **RWA override applies** (see §5) for real-world asset backing — use the RWA-specific rubric.

**Edge cases:**
- **Gold-backed stables** (PAXG, XAUT, IONau): differentiate allocated physical bullion (good: LBMA vaulted, regulated) from in-situ mineral claims (weak: depends on mining operator solvency, extraction timeline, valuation methodology).
- **Issuer going-concern flags**: if issuer has filed 10-K with substantial doubt re: going-concern (e.g., I-ON Digital 2026-04-16), A4 rises to at least band 7-8 regardless of nominal over-collateralization.
- **Proof-of-Reserves freshness**: Chainlink PoR is live but depends on attestor. If attestor is the same entity as issuer (e.g., self-attestation), score 1-2 bands worse.
- **Crypto-collateralized stables** (DAI, LUSD, USDS): backing = on-chain crypto. Verify over-collateralization ratio live via protocol dashboard. Rating scales with liquidation mechanism quality + collateral liquidity.
- **Fiat-backed with segregated custody + daily PoR** (e.g., Paxos with KPMG monthly + Chainlink PoR): band 1-2 territory.
- **Delta-neutral backing** (Ethena USDe, Resolv USR): A4 scores on hedge execution quality + insurance fund size. Cross-reference L1 C5/C4. Band 5-7 typical.
- **Tranche junior tokens** (jrUSDe, reUSDe): backing = underlying L2 asset minus senior tranche priority claim. Score A4 on effective junior backing + underlying asset's own A4. Band usually 1-2 bands worse than the senior.

**What to look for:**
- 100%+ live PoR + Big-4 monthly + regulated custody = band 1-2
- Over-collateralized on-chain + public dashboard = band 3-4
- Semi-annual attestation or some opaque reserves = band 5-6
- No attestation or known under-collateralization = band 9-10

---

## A5 — MARKET CAP / SUPPLY CONCENTRATION (Weight: 10%)

**Objective:** Quantify size and wallet-level supply concentration.

**Data source priority:**
1. **CoinGecko** for market cap + circulating supply
2. **Etherscan token holders tab** for top-100 wallet distribution
3. **Nansen / Arkham** for wallet labeling (identify CEX, institutional, DAO, foundation wallets)
4. **DefiLlama stablecoins / protocol TVL** for issuer-side supply

**Verification sequence:**
1. **Fetch current mcap** (circulating × price)
2. **List top-10 wallets** by holdings
3. **Label each wallet** (CEX hot/cold wallet, foundation treasury, DAO timelock, burn address, AMM LP, bridge contract)
4. **Calculate concentration** excluding structural holders (CEX hot wallets serving users, AMM LPs, canonical bridges)
5. **Net concentration**: % of non-structural top wallet holdings

**Critical checks:**
- **Structural holders don't count as concentration risk**: a Circle custody wallet holding 20% of USDC supply isn't concentration — it's the issuer's operational vault. Same for CEX hot wallets, Uniswap pools, canonical bridges.
- **Material holders** are: whales (individual non-labeled wallets), early investors, team treasury, unknown smart contracts holding significant supply.
- **Very small-cap assets** (<$10M mcap) score band 7-10 automatically — insufficient data to distinguish holder quality; liquidity risk dominates.

**Edge cases:**
- **Native assets** (ETH, BTC): mcap >$100B, distribution is structurally broad → band 1 by default.
- **Wrapped assets**: concentration of the wrapper (WETH held in DEX pools, WBTC held in DeFi protocols) is structural — don't count as concentration risk.
- **Newly-launched tokens** (<30 days): mcap data is noisy, founding team / airdrop recipients may concentrate supply artificially. Score band 7-8 until post-unlock distribution stabilizes.
- **Yield-bearing tokens** (sUSDe, pmUSD): holders are typically depositors/LPs. Low whale concentration is normal.

**What to look for:**
- Mcap >$10B + no single non-structural wallet >5% = band 1-2
- Mcap $100M-$1B + manageable concentration = band 5-6
- Mcap <$100M OR top non-structural wallet >20% = band 7-8
- Mcap <$10M or no supply data = band 9-10

---

## 5. RWA OVERRIDE FOR A4

For assets backed by **Real-World Assets** (tokenized T-bills, invoices, real estate, commodities, insurance treaties, mineral reserves), **replace the standard A4 rubric with the RWA-specific rubric** in `risk_methodology.md §3.3`.

Key differentiators:
- **Legal enforceability across jurisdictions** (multi-jurisdiction = 1-2; untested = 5-6; no legal claim = 9-10)
- **Liquidation timeline** (<30 days = 1-2; 60-90d = 5-6; >90d = 7-8)
- **Valuation cadence** (daily mark-to-market = 1-2; weekly = 3-4; monthly = 5-6; infrequent = 7-8)
- **Custodian jurisdiction** (US/EU/CH = 1-4; partial regulation = 5-6; offshore = 7-8)

Always combine with a **"flag cross-dependency with L1 C5"** note when backing IS the issuer's active operation (e.g., Ondo's US Treasury brokerage, Re Protocol's reinsurance treaty management).

---

## 6. NATIVE ASSET DEFAULTS

**ETH (native ETH and WETH on Ethereum):**
- A1 = 1 (no peg mechanism; native asset)
- A2 = 1 (no peg, no depeg possible)
- A3 = scored normally based on current market liquidity
- A4 = 1 (no collateral backing needed; asset itself)
- A5 = 1 (broad distribution, >$200B mcap)

**BTC (on its native chain):**
- Same defaults apply when scoring BTC-pegged wrapped tokens — the underlying BTC has A1/A2/A4/A5 = 1; the wrapper adds its own assessment layer.

**When WETH contract is specifically assessed:** same defaults as native ETH because WETH9 is immutable since 2017 with no upgrade path, guaranteeing 1:1 backing by locked ETH.

---

## 7. TRANCHE TOKENS (structured asset handling)

For tokens that represent **tranches of an underlying asset** (junior tranches absorbing first-loss, senior tranches with protection, locked/time-restricted variants):

Examples:
- **jrUSDe** (Strata junior tranche of USDe, first-loss)
- **reUSDe** (Re Protocol junior tranche, underwriting risk)
- **USD0++** (Usual locked 4-year variant of USD0)
- **sreUSDe / sUSDS / sDAI** (staked yield-bearing variants — these are typically NOT tranches, just yield-bearing wrappers. Distinguish from first-loss tranches.)

### 7.1 Classification decision

**Option A — treat as L2 asset (recommended):** Tranche tokens are standalone tradeable ERC-20s with their own peg/redemption dynamics. Score them as L2 assets with **explicit tranche-mechanism modifiers** in A1 and A4.

**Option B — treat as L3 strategy:** If the token is NOT independently tradeable (only redeemable direct from protocol) AND is purely a deposit-with-lockup wrapper, treat as L3 position.

Default to Option A unless the token lacks secondary market liquidity.

### 7.2 Scoring conventions for tranche tokens

**A1 (Peg Mechanism):** Inherit from underlying asset, but add **+1 to +3 band penalty** based on tranche position:
- Senior tranche with principal protection → +0 to +1 penalty
- Junior tranche with first-loss exposure → +2 to +3 penalty
- Locked/time-restricted (no active redemption during lock) → +1 additional penalty

**A2 (Depeg History):** Measure against the **tranche's own stated NAV**, not the underlying asset's peg. Capture specifically:
- Discount to NAV during stress periods
- Redemption-window-related price dynamics (e.g., reUSDe quarterly 72h window)
- Any first-loss event realized

**A3 (Liquidity Depth):** Tranche tokens typically have thinner liquidity than their underlying. Cap at best-case band 5-6 unless explicit evidence of deep liquidity ($10M+ swap at <0.5% slippage).

**A4 (Collateral Backing):** Score as underlying asset's A4 **minus senior tranche priority claim**. For junior tranches, this effectively means "what remains after senior claims are satisfied." Typically 1-2 bands worse than the senior.

**A5:** Score per standard rubric.

### 7.3 L1 cross-reference (dual-assessment)

Tranche tokens require **BOTH**:
- L1 of the **tranche issuer** (e.g., Strata for jrUSDe, Re Protocol for reUSDe, Usual for USD0++)
- L1 of the **underlying asset's issuer** (e.g., Ethena for USDe as input to jrUSDe)

Both L1s feed into any L3 strategy that uses the tranche token. Apply Multi-Protocol Rule (§4.2.2) when composing.

---

## 8. HARD CAP APPLICATION (L2-specific)

Per `risk_methodology.md §4.2.1`, an asset is **EXCLUDED** if ANY of:

1. **ARS > 6.5** *(v3.7: raised from 6.0; band 6.0–6.5 is now WATCHLIST)* — exclusion via weighted sum
2. **A2 ≥ 9** — exclusion via depeg history (sustained peg loss >15% OR >7d)

**L2 has 2 hard caps + 1 WATCHLIST band** (vs L1's 6 hard caps + 1 watchlist band):
- **ARS > 6.5** is a combinatorial threshold — multiple mid-range scores (7s and 8s) can push ARS over 6.5 even without any single 9+ trigger.
- **ARS 6.0–6.5 (WATCHLIST, v3.7)** — eligible with 15% per-vault cap, monthly re-scoring, documented monitoring triggers. Assets in this band have shown structural fragilities but functional exit paths (typically queue/cooldown async withdrawal); they are not mechanism-failed.
- **A2 ≥ 9** directly captures "mechanism has broken in the past." This is non-compensable: no amount of current liquidity or backing makes up for demonstrated peg failure >15% or >7d.

**Reassessment trigger rule:** if L2 reassessment degrades an asset to trigger either hard cap on an already-deployed strategy, **initiate exit within 7 days**. Coordinate with the L3 strategy registry to identify which strategies hold the asset. **WATCHLIST classification does NOT trigger forced unwind**, but does require allocation within the 15% per-vault cap (any over-cap excess must be unwound within 30 days of the rescore).

**Non-triggers (document but do NOT auto-exclude):**
- Single-criterion score of 7-8 (common for young assets) → document, apply concentration caps per Section 5
- High A5 (market cap small) without other red flags → apply position sizing limit, don't auto-exclude
- High A3 (thin liquidity) without depeg → apply position sizing + monitor (v3.7: with documented async exit path, A3 may receive credit per §A3)

---

## 9. ASSET RISK SCORE (ARS) — COMPOSITE CALCULATION

**Formula:** `ARS = 0.20·A1 + 0.25·A2 + 0.25·A3 + 0.20·A4 + 0.10·A5`

**Process:**
1. Score A1-A5 independently per rubrics above
2. Multiply each by its weight
3. Sum to 2 decimal places
4. Apply hard cap check (§8)
5. Document rationale for each criterion + sources

**Example (hypothetical stablecoin):**
- A1 = 3 (fiat-backed, well-attested)
- A2 = 5 (one moderate depeg event)
- A3 = 2 (deep liquidity)
- A4 = 2 (monthly Big-4 attestation)
- A5 = 3 (mid-cap)
- ARS = 0.20×3 + 0.25×5 + 0.25×2 + 0.20×2 + 0.10×3 = 0.60 + 1.25 + 0.50 + 0.40 + 0.30 = **3.05**

---

## 10. CROSS-LAYER COORDINATION

### 10.1 With Layer 1

For every L2 asset requiring dual-assessment (§0.2):
- Record the **L1 PRS** of the issuer in the L2 assessment
- If **L1 is excluded** (hard cap triggered), flag at top of L2: *"L1 issuer excluded — this L2 is produced for documentation only; L3 strategies using this asset inherit the exclusion cascade."*
- If **L1 PRS > 5.5** (concentration-capped), note the cascade impact on L3 position sizing

### 10.2 With Layer 3

L2 ARS feeds into L3 via:
- **Single-asset strategies:** AssetRisk = ARS directly
- **Paired strategies (LPs):** AssetRisk = 0.75·max(Ai) + 0.25·mean(Ai) per §4.2.3
- **Pendle PT strategies:** S2 sub-score mapped from ARS per §4.3.3 rubric
- **Lending vault strategies:** S2 sub-score mapped from weighted avg ARS of vault collateral per §4.3.4 rubric

### 10.3 Reassessment cadence

- **Monthly baseline:** recheck A2 (depeg monitoring), A3 (liquidity), A5 (mcap).
- **Quarterly:** full A1-A5 re-score.
- **Material-event triggers** (immediate reassessment):
  - Sustained deviation > 0.5% from peg for > 4h (A2 check)
  - Attestation missed / delayed > 7 days past cadence (A4)
  - Custodian change (A4)
  - Issuer going-concern filing (A4)
  - 30-day liquidity drop > 40% (A3)
  - Related L1 protocol incident (dual-assessment cascade)

---

## 11. MISSING DATA PROTOCOL (L2-specific)

### Case 1 — Non-disclosure as risk signal

If backing/custodian/attestation details are **not publicly disclosed**, score that criterion as if the worst plausible interpretation applied. Non-disclosure is not neutral; it's a risk signal per Missing Data Protocol Case 1.

Example: no Proof-of-Reserves feed + no third-party attestation → score A4 in band 7-8 minimum.

### Case 2 — Temporarily inaccessible data

If data source is down or rate-limited, apply Case 2 from `risk_methodology.md §4.2.5`: score the criterion at 7 (median-neutral), flag ⚠️ TEMPORARILY INACCESSIBLE, retry within 48 hours, re-score once resolved.

### Case 3 — Data source divergence ≥ 20%

Between two reputable sources (CoinGecko vs CoinMarketCap, DefiLlama vs issuer self-report), if values diverge by ≥20%:
- Use the **more conservative** (higher risk) source
- Document both and flag for CEO review
- Apply Case 3 from `risk_methodology.md §4.2.5`

### Case 4 — Asset too young for meaningful A2

Assets with <30 days of trading history cannot be fairly scored on A2 (depeg history). Apply:
- A2 = 7 (conservative baseline)
- Flag LIMITATION: "Insufficient history for A2 calibration"
- Monthly re-score until 90 days of history accumulate
- Retain cap on concentration until A2 stabilizes

---

## 12. OUTPUT CONTRACT

Every L2 assessment file should include:

1. **Header**: asset name, date, methodology version (v3.6), analyst, status
2. **Summary table**: A1-A5 scores + weighted values + ARS + verdict
3. **Scope note**: chain, wrapping, dual-assessment L1 reference
4. **Protocol/asset overview**: issuer, launch date, mcap, peg target
5. **Per-criterion rationale**: A1 through A5 with sources and verification evidence
6. **ARS calculation** with weighted breakdown
7. **Hard cap check**: ARS > 6.5 AND A2 ≥ 9 checks (v3.7 thresholds; ARS 6.0–6.5 = WATCHLIST band, eligible with 15% per-vault cap)
8. **Final decision**: APPROVED / EXCLUDED + concentration cap per Section 5 Rule 3
9. **Key strengths** and **Key risks** (3-6 bullets each)
10. **Sources**: primary + secondary + incident databases
11. **Notes / Flags / Limitations**: explicit about what was NOT verified, source divergence, cross-layer cascades
12. **Revision log** (if assessment supersedes a prior one)
13. **Final triggers**:
    ```
    ASSESSMENT_STATUS: COMPLETE
    PUBLISHER_TRIGGER: L2|<Asset Name>|<assessment_path>|<registry_path>
    ```

Registry update: append new entry to `registries/asset_registry.json` with full criteria breakdown + verdictNote + concentration cap.

---

**END OF METHODOLOGY**
