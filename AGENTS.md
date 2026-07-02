# SmartCredit Subagent Index

Machine-readable index of all 6 Claude Code subagents in `.claude/agents/`.
Each agent is a specialist that handles a specific Aave V3 liquidation risk task using the
SmartCredit Aave Liquidation Monitor MCP (`https://mcp.smartcredit.io/web3-borrow-lend-mcp`).

**Links:** [Website](https://smartcredit.io) · [GitHub](https://github.com/SmartCredit-io/web3-borrow-lend-mcp)

**Quick setup:**
```bash
claude mcp add --transport http smartcredit-aave-liquidation-monitor \
  https://mcp.smartcredit.io/web3-borrow-lend-mcp
```

No API key required — the endpoint is public.

---

## Agent Directory

### smartcredit-liquidation-monitor
**File:** `.claude/agents/smartcredit-liquidation-monitor.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `get_liquidation_risk`
**Purpose:** Core liquidation risk check for a single Aave V3 borrower position. Returns risk tier, liquidation probability, health factor, and per-asset liquidation prices.
**Triggers:** "check my Aave position", "what is my liquidation risk?", "will I get liquidated?", "health factor for 0x...", "am I safe on Aave?", "is this borrower at risk?"
**Input:** borrower wallet address (required); optional: hf_threshold, loan_term, sigma_window
**Output:** Tier badge, health factor, liquidation probability (% over loan_term days), per-asset liquidation prices, brief recommendation

---

### smartcredit-health-advisor
**File:** `.claude/agents/smartcredit-health-advisor.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `get_liquidation_risk`
**Purpose:** Analyzes an Aave V3 position and provides specific, prioritized advice on how to improve the health factor and reduce liquidation probability.
**Triggers:** "how do I improve my health factor?", "what should I do to avoid liquidation?", "advise on my Aave position", "help me de-risk my borrow", "how do I make my position safer?"
**Input:** borrower wallet address
**Output:** Risk summary + ranked list of actionable steps (add collateral, repay debt, which asset has most impact) with estimated HF improvement

---

### smartcredit-threshold-finder
**File:** `.claude/agents/smartcredit-threshold-finder.md`
**Model:** claude-haiku-4-5-20251001
**Tools:** `get_liquidation_risk`
**Purpose:** Extracts and presents all per-asset liquidation price thresholds. Tells the user exactly what price drop triggers liquidation for each collateral asset.
**Triggers:** "what price will liquidate my position?", "find my liquidation prices", "what ETH price triggers liquidation?", "price thresholds for 0x...", "how far can ETH drop before I get liquidated?"
**Input:** borrower wallet address
**Output:** Table: asset → current price (USD) → liquidation price (USD) → % price drop to liquidation

---

### smartcredit-stress-tester
**File:** `.claude/agents/smartcredit-stress-tester.md`
**Model:** claude-sonnet-4-6
**Tools:** `get_liquidation_risk`
**Purpose:** Stress tests a position across multiple time horizons (7, 14, 30, 90 days) by calling `get_liquidation_risk` four times with different `loan_term` values. Shows how liquidation risk evolves over time.
**Triggers:** "stress test my Aave position", "what is my 30-day liquidation risk?", "how does my risk change over time?", "model different scenarios for my position", "risk over 90 days"
**Input:** borrower wallet address
**Output:** Risk matrix table: loan_term (days) → liquidation probability → tier; plus interpretation of risk trend

---

### smartcredit-portfolio-scanner
**File:** `.claude/agents/smartcredit-portfolio-scanner.md`
**Model:** claude-sonnet-4-6
**Tools:** `get_liquidation_risk`
**Purpose:** Scans multiple Aave V3 borrower positions in sequence and produces a portfolio-level risk summary ranked by liquidation probability.
**Triggers:** "scan these wallets for liquidation risk", "check all my positions", "portfolio risk on Aave", "which position is most at risk?", "rank these addresses by liquidation probability"
**Input:** list of borrower wallet addresses
**Output:** Ranked table (highest risk first): address → health factor → liquidation probability → tier; overall portfolio risk assessment; urgent flags for Danger/Critical positions

---

### smartcredit-risk-reporter
**File:** `.claude/agents/smartcredit-risk-reporter.md`
**Model:** claude-sonnet-4-6
**Tools:** `get_liquidation_risk`
**Purpose:** Generates a full written risk report for a borrower position — suitable for sharing with a team, saving as documentation, or inclusion in a risk management workflow.
**Triggers:** "generate a risk report for 0x...", "write a liquidation report", "I need a formal risk assessment of my Aave position", "document my position risk", "create a risk summary I can share"
**Input:** borrower wallet address; optional: loan_term, sigma_window for model parameters
**Output:** Structured markdown report: executive summary, position details, risk assessment, collateral analysis, debt breakdown, liquidation scenarios, recommendations
