# Layer 1 — Protocol Risk Assessment Methodology

**Version:** v3.6
**Date:** 2026-04-23
**Companion to:** `risk_methodology.md` (canonical source for rubrics, formulas, and hard caps)
**Amendment applicability:** Incorporates v3.5 (C4 max rule) and v3.6 (C5 cross-chain + C1 audit coverage) amendments

**Purpose:** Practical guide for scoring Layer 1 (protocol-level) risk. Contains data sources, verification steps, and application notes for each criterion.

**For scoring rubrics and formulas, see `risk_methodology.md`.**

---

## C1 — AUDIT STATUS (Weight: 25%)

**Objective:** Evaluate audit quality, recency, findings resolution, and bug bounty programs.

**Upgradeability pre-check (required before scoring):**

Classify core contracts before applying any rubric:

- **Type A (immutable):** Core logic is non-upgradeable. Score on audit quality and UCD. Audit age NOT penalized.
- **Type B (upgradeable):** Core contracts use a proxy or admin upgrade key. Full rubric with recency.
- **Mixed:** Apply Type A to immutable components, Type B to upgradeable components. Use the more conservative (higher) score.

**Unaudited Code Delta (UCD):** Estimated % of deployed code (by functional scope) added or modified since the most recent audit. Includes new integrations, wrappers, reward routing, and post-audit additions.

**Data source priority (1 > 2 > 3):**
1. Protocol security page (e.g., `protocol.com/security`)
2. GitHub repository `/audits` or `/security` directory
3. Audit firm websites (Trail of Bits, OpenZeppelin, Certora, Spearbit, ChainSecurity, Cantina)
4. Immunefi or HackerOne bug bounty pages

**Verification sequence:**
1. **Count audits:** Identify all security audits (exclude internal reviews)
1b. **Identify upgradeability:** Classify core contracts as Type A (immutable) or Type B (upgradeable) before applying rubric — check Etherscan proxy tab, deployment docs, and audit scope notes
2. **Verify firms:** Cross-reference auditor names with top-tier list in risk_methodology.md
3. **Check recency:** Note date of most recent audit (Type B only — penalized in rubric; Type A — record for UCD estimation)
3b. **(Type A only) Estimate UCD:** Enumerate integrations, wrappers, reward contracts, or other code additions deployed since most recent audit; estimate as % of total functional scope
4. **Read findings:** Navigate to "Findings" or "Issues" section in audit reports
5. **Flag unresolved issues:** Look for High/Critical severity with status "Acknowledged" or "Won't Fix"
6. **Check bug bounty:** Verify active program amount (if present)
7. **Verify formal verification:** Check if mathematical proofs exist for core logic

**Critical verification step:**
Reading audit **findings sections** is mandatory, not just counting audits. "Acknowledged/Won't Fix" High/Critical findings are red flags that can elevate C1 score even with many audits.

**Edge cases:**
- **If PDFs inaccessible:** If data is inaccessible, apply Missing Data Protocol Case 2 from SKILL (score 7, flag ⚠️ TEMPORARILY INACCESSIBLE, retry within 48 h)
- **Preliminary vs. final reports:** Only count final reports; don't double-count
- **Audit scope:** Verify audit covers all production contracts, not just a subset
- **Post-upgrade audits:** Check if major version upgrades have been re-audited

**What to look for:**
- Top-tier firms: Trail of Bits, OpenZeppelin, Certora, Spearbit, ChainSecurity, Cantina
- Bug bounty thresholds: ≥$1M (excellent), $250K-$1M (good), <$250K (moderate)
- Formal verification: Certora, Runtime Verification, or similar proofs

---

## C2 — TVL HISTORY (Weight: 10%)

**Objective:** Assess protocol size, stability, and trajectory.

Data source priority (1 = primary, use first; fall back in order):
  1. DefiLlama API (api.llama.fi/protocol/[slug])
  2. Protocol dashboard TVL display
  3. L2Beat (for L2 protocols), Token Terminal
  4. DeFi Pulse (cross-reference only)

**Verification sequence:**
1. **Current TVL:** Get latest USD value
2. **30-day trend:** Compare TVL today vs. 30 days ago, calculate % change
3. **All-time high (ATH):** Identify max TVL and date
4. **Current vs. ATH:** Calculate current TVL as percentage of ATH
5. **Trend analysis:** Plot 30-day chart to assess stability vs. volatility

**Key metrics to extract:**
- Current TVL (USD)
- TVL 30 days ago (USD)
- % change 30-day
- ATH TVL (USD) and date
- Current as % of ATH

**Edge cases:**
- **Source divergence ≥20%:** If sources diverge ≥20%, apply the Missing Data Protocol Case 3 from `risk_methodology.md` Section 4.2.5.
- **Missing historical data:** For new protocols (<3 months), note in LIMITATIONS, use available data range
- **Multi-chain protocols:** Sum TVL across all chains, note if one chain dominates (>80%)

**What to look for:**
- Steady growth or stability indicates healthy adoption
- Sharp drawdowns (>50% in 30 days) are red flags
- Distance from ATH indicates whether protocol is recovering or declining from peak

---

## C3 — GOVERNANCE QUALITY (Weight: 20%)

**Objective:** Evaluate upgrade controls, timelocks, multisig security, admin key risks.

**Data source priority:**
1. **On-chain verification (preferred):** Call governance contracts via RPC
   - `owner()` / `getOwners()` / `getThreshold()` / `delay()`
2. **Safe API:** For multisig details (signers, threshold)
3. Protocol documentation (governance page, security docs)
4. Audit reports (often document governance architecture)

**Verification sequence (MANDATORY — applies to ALL economic-primitive admin roles, not only upgrades):**

Assessor MUST perform on-chain reads for every critical role listed below. Reading audit reports (for role documentation) is mandatory, not optional.

1. **Identify upgrade mechanism:** Proxy pattern, DAO governance, multisig, or immutable
2. **Verify timelock:** If present, check delay duration (on-chain preferred)
3. **Verify multisig:** If present, check threshold, total signers, and signer identities
4. **Check governance audit:** Verify governance contracts themselves are audited
5. **Verify admin keys for ALL economic-primitive roles (not just upgrades):**
   - `owner()` / `admin()` / `proxyAdmin()` — upgrade authority
   - `minter()` / `MINTER_ROLE` — token mint authority
   - `burner()` / `BURNER_ROLE` — token burn authority
   - `pauser()` / `PAUSER_ROLE` — user-fund freeze authority
   - `priceSetter()` / oracle-writer / rate-setter — share-price / NAV / redemption-ratio control
   - `redemptionController()` / `REDEMPTION_ROLE` — redemption-rule modifier
   - `blacklister()` — account-freeze authority (stablecoins)

   For EACH role: is the holder an EOA, a multisig (which threshold?), or a KMS-backed single key? Apply the **multisig-threshold-floor rule (CEO decision 2026-04-25)**:

   - **Multisig acceptable without timelock if EITHER:**
     - **(a) threshold ratio ≥ 60%** (e.g., 3/5, 4/7 = 57% rounded up via discretion if signers doxxed, 5/7, 4/5), **OR**
     - **(b) threshold absolute ≥ 4 distinct parties AND threshold ratio ≥ 40%** (e.g., 4/8 = 50% with 4-party collusion barrier — Kinetiq pattern, 5/9 = 56%, 4/10 = 40%)

     Rationale: A multisig protects against single-point-of-failure attacks. The risk it controls is *collusion or compromise of enough signers to act unilaterally on drain-vector roles*. Both ratio and absolute count matter:
     - **Ratio ≥ 60%** (clause a) gives proportional comfort that a majority of signers must agree
     - **Absolute ≥ 4 distinct parties** (clause b) gives substantive collusion-barrier comfort even at lower ratios — 4 separate parties compromised simultaneously is a high bar in practice; lower ratio (≥ 40%) is acceptable when paired with this absolute threshold

     Timelock remains best practice and recommended (48h+ = excellent) regardless. Industry standard: pause / emergency-response roles often intentionally omit timelock for fast incident response — methodology accepts this under either acceptance clause.

   - **Multisig fails BOTH (a) AND (b)** on a critical role → **requires timelock ≥ 24h**. Without timelock → **C3 = 9 automatic trigger → EXCLUDED**.

     Specifically disqualified without timelock:
     - 3/7 = 43% ratio + 3 absolute (Dinero pattern, fails both — 43% < 60% AND 3 abs < 4)
     - 3/8 = 37.5% ratio + 3 absolute (Alchemix Dev Multisig pattern, fails both)
     - 2/5 = 40% ratio + 2 absolute (Treehouse pattern, fails both)
     - 2/3 = 67% ratio + 2 absolute (passes (a) on ratio but small absolute count is a concentration concern; assessor judgment applied — recommended to require timelock if signers < 3 distinct entities)
     - Single EOA on drain-vector role (always fails — no multisig at all)

     Rationale for the ≥ 4 absolute floor in clause (b): 4 separate parties required for collusion is a substantive barrier even if the % ratio is lower; this preserves the methodology's intent (catching effectively-single-key patterns and small-multisig vulnerabilities) without over-penalizing wider-N multisigs that still require many parties to act.

   - **Single EOA OR KMS-single-key (e.g., AWS-KMS, single-device MPC)** on a critical *drain-vector* role (mint, burn, upgrade, price-setter, redemption-rule, blacklister) → **C3 = 9 automatic trigger → EXCLUDED** regardless of timelock (single-key compromise = single-point-of-failure regardless of delay window).

   - **Single EOA on a *DoS-vector* role (pause only, when pause cannot drain or upgrade)** → noted as a residual concern in the assessment, but **not auto-disqualifying**. This is a defensible industry-standard pattern (Aave Guardian, Compound pause guardian, Re Protocol PAUSER on ICL) — fast emergency-response prioritized over multisig coordination overhead. Pause has a DoS blast radius but cannot drain user funds.

**Critical checks:**
- **Timelock duration:** ≥48h (excellent), 24-48h (good), 12-24h (moderate), <12h (weak)
- **Multisig threshold:** ≥4/7 or 5/7 (excellent), 3/5 (good), 2/3 (weak), 2/2 (very weak)
- **Signer identities:** Doxxed (excellent), known pseudonyms (good), anonymous (weak)
- **Immutability:** Core logic immutable with peripheral upgradeable (best for mature protocols)
- **Economic-primitive role separation:** Even with multisig governance on upgrades, if mint/price/redemption roles are held by EOAs/KMS single keys, the multisig provides no protection against the most common drain vectors.

**Leading-indicator worked examples (pre-hack structural red flags the methodology must catch):**

| Case | Pre-hack red flag | Methodology response |
|---|---|---|
| **Resolv (March 2026 $23-25M exploit)** | AWS-KMS single-key held mint + share-price-setter roles for USR | Under step 5 above: EOA/KMS single-key on mint OR price-setter (drain-vector roles) → C3 = 9 → EXCLUDED *before* the exploit |
| **Kelp (April 2026 $292M Lazarus exploit)** | 1-of-1 LayerZero DVN on rsETH bridging | v3.6 C5 cross-chain rule: DVN threshold < 2-of-N → C5 = 9 → EXCLUDED *before* the exploit |
| **Usual / USD0++ (Jan 2025 $20M cascade)** | Redemption-rule modifier held by **small (< 3/5) multisig without timelock** — unilateral parameter flip executed | Under multisig-threshold-floor rule: multisig < 3/5 without timelock on redemption-rule role → C3 = 8-9 → EXCLUDED *before* the incident. (If Usual had been 3/5+ multisig, methodology would treat the structural barrier as sufficient — the historical incident demonstrates the < 3/5 case.) |

The objective is identification **before** the hack, not confirmation **after**. An assessor who scores a protocol < 9 on C3 without verifying every critical role on-chain is assessment-malpractice.

**Emergency multisig handling:**

If a protocol has a secondary emergency multisig with a shorter timelock than the primary governance mechanism:

1. Score C3 based on the **PRIMARY** governance mechanism (standard rules)
2. Apply a **+1 penalty** to the C3 score for the emergency path if the emergency multisig threshold is < 3/5 or timelock < 6h
3. **Cap the penalty at +2** regardless of emergency path severity
4. **Document both mechanisms explicitly** in the assessment

*Rationale: emergency paths are legitimate engineering choices. The penalty reflects the additional attack surface, not automatic disqualification.*

**Edge cases:**
- **Multi-stage governance:** Some protocols have proposal → voting → timelock → execution. Document all stages.
- **Emergency admin:** Some protocols have emergency multisig with shorter timelock — apply the emergency multisig handling rules above.
- **Operational security note:** HSM usage, key rotation, access controls are assessed qualitatively (not scored in C3 but may trigger exclusion if known to be weak)

**What to look for:**
- On-chain DAO with long timelock on ALL economic-primitive roles (upgrade + mint + price + redemption) = best
- Known multisig with ≥4 signers protecting every critical role = good
- Anonymous or low-threshold multisig = red flag
- **Single EOA / KMS single-key on ANY of {upgrade, mint, pause, price-setter, redemption-rule} = immediate C3 = 9 exclusion trigger** (do not wait for the protocol to use that authority maliciously — the structural capability alone is disqualifying)

---

## C4a — INCIDENT HISTORY (Weight: 14% of total, 70% of C4)

**Objective:** Identify exploits, assess response quality.

**Data source priority:**
1. **Incident databases:** rekt.news, DeFiYield REKT, DeFiLlama hacks
2. **Protocol announcements:** Blog, Twitter/X, governance forum
3. **Bug bounty platforms:** Immunefi, HackerOne (for white-hat rescues)

**Verification sequence:**
1. **Search incident databases:** Look up protocol name
2. **Check protocol communications:** Search blog/Twitter for "exploit", "incident", "post-mortem"
3. **Classify incidents:** Amount lost, compensation status, root cause
4. **Assess response:** Post-mortem quality, fix deployment speed, user communication

**Incident classification:**
- **Amount:** <$1M (minor), $1M-$5M (moderate), $5M-$50M (major), >$50M (catastrophic)
- **Compensation:** Full (100%), Substantial (>75%), Partial (25-75%), Minimal (<25%), None
- **Response quality:** Post-mortem published? Fix deployed? Root cause addressed?

**Edge cases:**
- **White-hat interventions:** If bug found and fixed before exploit, counts as minor incident with full recovery
- **Repeated incidents:** Multiple exploits (even if small) indicate systemic issues → elevate C4a
- **No post-mortem:** Lack of transparency after incident is a red flag

**What to look for:**
- Zero exploits + active bug bounty ≥$1M = best
- Minor incident with full compensation + rapid response = acceptable
- Major incident with incomplete compensation = red flag
- Repeated incidents = pattern of weak security

---

## C4b — OPERATIONAL AGE (Weight: 6% of total, 30% of C4)

**Objective:** Assess battle-testing duration.

**Data source priority:**
1. DefiLlama first TVL datapoint (indicates mainnet launch with usage)
2. Protocol announcement (mainnet launch date)
3. GitHub first commit on mainnet contracts (deployment date)

**Verification sequence:**
1. Check DefiLlama protocol page for "First TVL date"
2. Cross-reference with protocol announcement or blog post
3. If discrepancy, use earlier date (more conservative)
4. Calculate months operational as of assessment date

**Edge cases:**
- **Testnet vs. mainnet:** Only count mainnet operational time
- **Multi-chain:** Use earliest mainnet deployment across chains
- **Forks:** If protocol is a fork of battle-tested code (e.g., Compound V2 fork), note this but don't transfer the original's age

**What to look for:**
- >3 years = excellent (survived multiple market cycles)
- 2-3 years = good (survived at least one cycle)
- 1-2 years = moderate (one full year of operation)
- <6 months = weak (insufficient battle-testing)

---

## C4 COMPOSITE CALCULATION

**Formula:** `C4 = 0.70 × C4a + 0.30 × C4b`

**Process:**
1. Score C4a independently (incident history)
2. Score C4b independently (operational age)
3. Calculate composite: `C4 = 0.70 × C4a + 0.30 × C4b`
4. Round to 2 decimal places

**Example:**
- C4a = 2 (zero exploits, bug bounty ≥$1M)
- C4b = 4 (2-3 years operational)
- C4 = 0.70 × 2 + 0.30 × 4 = 1.40 + 1.20 = **3.60**

## C1 AUDIT COVERAGE GAP RULES (v3.6 amendment)

**Rule applies to all L1 assessments.** Elevates C1 score when audit coverage has systematic gaps:

| Coverage gap | C1 minimum |
|--------------|------------|
| UCD (Undocumented Critical Dependencies) > 50% of critical deployed surface | 7 |
| Major production layer (bridge adapter, vault, oracle, custody layer) entirely unaudited | 8 |
| Production-critical upgrade (>20% code change) without post-upgrade audit > 6 months | 7 |
| Audit found CRITICAL/HIGH finding on production contract, ACKNOWLEDGED-not-fixed > 90 days | 7 |

**Kelp validation:** Pre-incident C1=7 partially captured audit gap (UCD on DVN layer) but did not reach C1≥9 hard cap. v3.6 correctly does NOT force hard cap on C1 coverage gap alone (still requires C5 or C3 complementary trigger for EXCLUDED).

**Rationale:** Audit count alone is not sufficient. A protocol with 10 audits covering 50% of deployed code has the same risk profile as a protocol with 5 audits covering 100%. v3.6 makes this explicit.

## C4 HARD CAP RULE (v3.5 amendment)

**Rule:** Protocol is EXCLUDED if `max(C4a, C4 composite) ≥ 8`.

This is the **v3.5 amendment** to the prior v3.4 rule (which only checked `C4 composite ≥ 8`). The change prevents catastrophic incidents from being diluted below the hard cap threshold by long operational age.

**Worked example — why the amendment matters:**

Kelp DAO (April 2026):
- C4a = 9 (catastrophic $292M exploit)
- C4b = 4 (2 years operational)
- C4 composite = 0.70·9 + 0.30·4 = 7.50
- **v3.4 rule (composite ≥ 8):** 7.50 < 8 → NOT triggered (false negative)
- **v3.5 rule (`max(C4a, C4) ≥ 8`):** `max(9, 7.50) = 9` → TRIGGERED ✓

C4a alone is the dominant trigger; age cannot compensate a catastrophic incident.

**Monitoring:** Any C4a ≥ 7 is near-threshold; quarterly review recommended. C4a ≥ 8 is immediate exclusion.

See `methodology/v35_amendment.md` for full rationale, backtest, and migration path.

---

## C5 — SMART CONTRACT RISK (Weight: 15%)

**Objective:** Evaluate code complexity, dependencies, upgradeability, verification, and cross-chain configuration (v3.6).

### Part A: Cross-chain configuration hard caps (v3.6 — applied FIRST)

For any protocol with cross-chain bridge/messaging dependencies (LayerZero OFT, Wormhole, Axelar, Chainlink CCIP, Hyperlane, etc.), the following elevate C5 mandatorily, **before** the base complexity rubric is applied:

| Configuration | C5 minimum |
|---------------|------------|
| **1-of-1 DVN / single verifier** (e.g., LayerZero Labs as sole verifier) | **9 (hard cap → EXCLUDED)** |
| DVN threshold of 1-of-N or outside governance perimeter | 8 (watchlist) |
| DVN threshold < 2-of-N on critical asset transport | 8 |
| Bridge configuration undisclosed after mandatory verification | 8 |

**Compliant configurations (C5 floor unchanged — fall through to Part B):**
- Wormhole Guardian set ≥13-of-19 (industry default)
- Chainlink CCIP dual-committee verification
- LayerZero with ≥2 independent required DVNs + optional DVN threshold ≥2
- Axelar ≥4-of-N independent validators

**Kelp validation example:**
- Pre-incident: Kelp rsETH LayerZero OFT = 1-of-1 DVN (LayerZero Labs sole verifier)
- v3.4 score: C5=7 (off-chain dependency floor only)
- **v3.6 score: C5=9 → hard cap → EXCLUDED months before $292M exploit**

**Mandatory data points for cross-chain L1 assessments (v3.6):**
1. DVN threshold (X-of-Y format) — verified on-chain, not just documented
2. Verifier identity disclosure
3. Bridge adapter audit coverage status
4. Emergency pause capability on cross-chain surface
5. Governance perimeter coverage of DVN config changes

### Part B: Base complexity rubric (applies when Part A does not elevate the score)

**Data source priority:**
1. GitHub repository (public/private, LOC count, commit history)
2. Etherscan/block explorer (source verification status)
3. Protocol documentation (architecture diagrams, dependency list)

**Verification sequence:**
1. **Find repo:** Locate GitHub repository URL
2. **Verify source:** Check Etherscan "Contract" tab for verified source code
3. **Count LOC:** Estimate lines of code across all core contracts
4. **Identify dependencies:** List external protocols, oracles, libraries
5. **Check upgradeability:** Identify proxy pattern (transparent, UUPS, beacon, or immutable)
6. **Assess off-chain dependencies:** Look for keepers, relayers, backend APIs in docs

**Operational proxy for complexity assessment (not a rubric override):**
- **Simple:** <1000 LOC, minimal dependencies (e.g., basic lending protocol)
- **Moderate:** 1000-5000 LOC, standard dependencies (Chainlink, OpenZeppelin)
- **High:** >5000 LOC, novel patterns, multiple external integrations
- **Very high:** >10000 LOC, experimental architecture, heavy off-chain reliance

**Critical off-chain dependency rule:**
Protocols with critical dependencies on off-chain infrastructure (CEX accounts for delta hedging, cloud services for core operations) score **minimum 7** in C5, as such dependencies introduce counterparty and operational risks not mitigated by smart contract audits.

**Edge cases:**
- **Unverified source:** Immediate red flag → score 9-10
- **Closed source:** Immediate exclusion trigger
- **Audited dependencies:** If protocol uses audited libraries (OpenZeppelin, Solmate), this reduces risk

**What to look for:**
- Verified source on Etherscan = minimum requirement
- Simple, battle-tested architecture = best
- Novel patterns + audited + operational > 1 year = acceptable
- Unverified or experimental = red flag

---

## C6 — TEAM & TRANSPARENCY (Weight: 10%)

**Objective:** Assess founder doxxing, track record, communication quality.

**Data source priority:**
1. Protocol website "Team" or "About" page
2. LinkedIn profiles of founders
3. GitHub commit history (contributor activity)
4. Twitter/X public presence
5. Conference appearances, podcast interviews

**Verification sequence:**
1. **Identify founders:** List core team member names
2. **Verify doxxing:** Check if real names and photos are public
3. **Check LinkedIn:** Verify work history, education, prior projects
4. **Assess track record:** Look for prior successful DeFi projects
5. **Evaluate communication:** Check Twitter/Discord activity, blog post frequency
6. **Check entity registration:** See if company is registered (US, EU, etc.)

**Doxxing levels:**
- **Fully doxxed:** Real names, photos, LinkedIn, verifiable background
- **Partially doxxed:** Some team doxxed, some pseudonymous
- **Known pseudonyms:** Strong reputation, verifiable track record (e.g., Twitter followers, GitHub stars)
- **Anonymous:** No verifiable identity or track record

**Edge cases:**
- **Founder reputation (bounded override):** A founder's verified track record on a successful prior project (> 2 years operational, > $10M TVL at peak, no major incident) may improve the C6 score by a **maximum of one band (2 points)**. This override:
  - Cannot bring C6 below **3** (minimum score for partially doxxed)
  - Does **not** apply if the prior project ended in exploit or rug
  - Must be **documented explicitly** in the assessment rationale
  - Does **not** override the C6 ≥ 8 hard cap exclusion rule
- **Communication quality:** Active, transparent communication (weekly updates, clear roadmap) can offset partial anonymity
- **Entity registration:** Registered company in regulated jurisdiction adds credibility

**What to look for:**
- Fully doxxed team with multi-year DeFi track record = best
- Known pseudonyms with strong reputation = acceptable
- Anonymous with weak communication = red flag
- History of bad actors (e.g., rug pulls) = immediate exclusion

---

**END OF METHODOLOGY**
