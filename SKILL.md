---
name: smartcredit-aave-liquidation-monitor
version: 1.0.0
license: MIT
description: "Use this skill whenever a user asks about Aave V3 liquidation risk, health factor, collateral safety, borrow position monitoring, or liquidation price thresholds on Ethereum mainnet. Triggers on questions like: is my position safe?, will I get liquidated?, what is my health factor?, check my Aave position, what price triggers my liquidation?, how close am I to liquidation?, what is my liquidation probability?, stress test my borrow position, what ETH price will liquidate me?, advise on my Aave collateral, how do I improve my health factor?, scan my portfolio for liquidation risk, generate a risk report for my Aave position, what is the collateral ratio for 0x...?, is this borrower at risk of liquidation?, monitor this wallet on Aave, check Aave V3 borrower position, evaluate DeFi lending risk for this address, or any request to assess the liquidation probability, health factor, or collateral thresholds of an Aave V3 borrower on Ethereum."
metadata:
  openclaw:
    requires:
      env: []
    data_handling:
      external_endpoints:
        - url: https://mcp.smartcredit.io/web3-borrow-lend-mcp
          transport: HTTP (Streamable HTTP, MCP spec 2025-03-26)
          purpose: Aave V3 liquidation risk scoring for Ethereum mainnet borrower positions
          data_sent:
            - Borrower wallet addresses (pseudonymous on-chain identifiers)
            - Optional model parameters (hf_threshold, loan_term, sigma_window)
          data_NOT_sent:
            - Names, emails, or any off-chain PII
            - Private keys or seed phrases
            - Raw transaction data
          retention: Governed by SmartCredit's privacy policy
          privacy_policy: https://smartcredit.io/privacy

    emoji: 🏦
    homepage: https://github.com/SmartCredit-io/web3-borrow-lend-mcp
    author: SmartCredit
    tags:
      - aave
      - defi
      - liquidation
      - lending
      - health-factor
      - ethereum
      - web3
      - risk
      - mcp
      - borrow
---

# SmartCredit Aave Liquidation Monitor MCP

## What This Skill Does

The **SmartCredit Aave Liquidation Monitor MCP** connects any AI agent to real-time Aave V3 liquidation risk intelligence for Ethereum mainnet borrower positions. It uses a log-normal volatility model to compute liquidation probability, per-asset price thresholds, and tiered risk labels — from a single MCP tool call.

1. **Liquidation Risk Score** — compute liquidation probability for an Aave V3 position over a configurable time horizon
2. **Health Factor** — fetch the live health factor from Aave V3 contracts
3. **Per-Asset Liquidation Prices** — exact price level for each collateral asset that would trigger liquidation
4. **Risk Tier** — Safe / Watch / Warning / Danger / Critical / Liquidatable
5. **Stress Testing** — evaluate risk across multiple time horizons (7, 14, 30, 90 days)
6. **Portfolio Scanning** — batch-evaluate multiple borrower addresses

No API key required. The endpoint is public.

**MCP Server URL:** `https://mcp.smartcredit.io/web3-borrow-lend-mcp`
**GitHub:** https://github.com/SmartCredit-io/web3-borrow-lend-mcp
**Website:** https://smartcredit.io

---

## Capabilities

- **Liquidation Risk Scoring** — probability that a position will be liquidated within a configurable look-ahead window using a log-normal volatility model
- **Health Factor Monitoring** — live on-chain HF from Aave V3 contracts via RPC
- **Per-Asset Price Thresholds** — exact liquidation price for each collateral token
- **Stress Testing** — run the model across multiple time horizons to see how risk evolves
- **Portfolio Scanning** — evaluate multiple addresses in sequence and rank by risk

---

## When to Use This Skill

- User asks about liquidation risk, health factor, or collateral safety on Aave
- User wants to know what price will trigger their liquidation
- User needs advice on how to improve their health factor
- User wants to stress test a borrow position across different time horizons
- User is building a DeFi risk tool, lending platform, or position monitor
- User wants to scan multiple positions and rank them by liquidation probability

## When NOT to Use This Skill

- User asks about general Aave interest rates or yields → use Aave's own documentation or a DeFi data API
- User asks about protocols other than Aave V3 on Ethereum → this tool covers Aave V3 mainnet only
- User wants real-time price data or market cap → use a market data API (CoinGecko, etc.)
- User wants to analyze smart contract code → use a code auditing tool
- For full DeFi fraud screening of a borrower wallet → combine with ChainAware MCP fraud tools

---

## Supported Networks

Ethereum mainnet — Aave V3 only.

No other networks or lending protocols are currently supported.

---

## Step-by-Step Workflow

### For a single position risk check

1. **Confirm inputs** — borrower wallet address (required). All other parameters have sensible defaults.
2. **Call `get_liquidation_risk`** with the wallet address. Use defaults unless the user specifies otherwise.
3. **Check the response code** — 204 means the position's HF is above `hf_threshold` (safe by default definition). 200 returns the full risk profile.
4. **Interpret `liquidation_probability`** using the tier table below.
5. **Surface per-asset liquidation prices** from the `collateral[]` array.
6. **Report** tier, probability, health factor, and key thresholds in plain language.

### For finding liquidation price thresholds

1. Call `get_liquidation_risk` with the wallet address.
2. Extract `collateral[]` — each entry has `symbol`, `price_usd`, and `liquidation_price_usd`.
3. Compute `% drop = ((price_usd - liquidation_price_usd) / price_usd) × 100` for each asset.
4. Report a table: asset → current price → liquidation price → % drop to liquidation.

### For stress testing across time horizons

1. Call `get_liquidation_risk` four times with `loan_term` = 7, 14, 30, 90 (keep `sigma_window` fixed at 30).
2. Extract `liquidation_probability` and `tier` from each response.
3. Build a risk matrix table showing how probability and tier change over time.

### For portfolio scanning (multiple addresses)

1. Loop through the list of wallet addresses.
2. Call `get_liquidation_risk` for each address.
3. Collect `(address, health_factor, liquidation_probability, tier)` from each response.
4. Sort by `liquidation_probability` descending (highest risk first).
5. Report a ranked table and call out any positions in Danger or Critical tier.

---

## Risk Tier Thresholds

| Tier | Liquidation Probability | Recommended Action |
|---|---|---|
| Safe | 0–5% | No action needed |
| Watch | 5–15% | Monitor closely |
| Warning | 15–35% | Consider reducing exposure |
| Danger | 35–70% | Reduce position urgently |
| Critical | 70–100% | Immediate action required |
| Liquidatable | HF < 1.0 | Position is liquidatable now |

---

## Available Tools

### 1. `get_liquidation_risk` — Liquidation Risk Scoring

Fetches live on-chain Aave V3 data and runs a log-normal volatility model to compute liquidation probability.

**Inputs:**
- `user_address` (string, required) — borrower wallet address (`0x…`)
- `platform` (string, optional, default: `AAVE`) — only `AAVE` is currently supported
- `hf_threshold` (float, optional, default: `1.5`) — only score positions with HF below this value; returns 204 (safe) otherwise
- `loan_term` (int, optional, default: `7`) — model look-ahead in days (1–365)
- `sigma_window` (int, optional, default: `30`) — days of price history for volatility estimation (7–365)

**Key output fields:**
- `health_factor` — live Aave V3 health factor
- `liquidation_probability` — percentage probability of liquidation within `loan_term` days
- `tier` — Safe / Watch / Warning / Danger / Critical / Liquidatable
- `collateral[]` — per-asset: `symbol`, `price_usd`, `liquidation_price_usd`, `liquidation_threshold`
- `debt[]` — per-asset: `symbol`, `balance`, `price_usd`, `value_usd`

**Response codes:**
- `200` — full risk profile returned
- `204` — position is safe (HF >= hf_threshold); no body

**Example prompts that trigger this tool:**
- *"What is my liquidation risk on Aave for 0xABC...?"*
- *"Is this position safe? Wallet: 0xDEF..."*
- *"What ETH price would liquidate 0x123...?"*
- *"Check the health factor for this borrower."*
- *"Stress test this position over 30 days."*

---

### 2. `check_api_health` — Server Health Check

Pings the MCP server. No parameters. Returns `"ok"` or an error description.

**Example prompts:**
- *"Is the SmartCredit MCP server online?"*
- *"Check if the liquidation monitor is reachable."*

---

## Validation Checkpoints

### Input Validation
- ✅ `user_address` is provided and non-empty
- ✅ Address is a valid Ethereum address format (`0x` followed by 40 hex characters)
- ✅ `loan_term` is in range 1–365 if provided
- ✅ `sigma_window` is in range 7–365 if provided
- ⚠️ If the user specifies a non-AAVE platform, inform them that only Aave V3 is currently supported
- ⚠️ If the user asks about a non-Ethereum network, clarify that only Ethereum mainnet is supported

### Output Validation
- ✅ Handle 204 responses gracefully: report "position is safe (HF above threshold)"
- ✅ If `health_factor < 1.0`, flag immediately as Liquidatable — highest urgency
- ✅ Surface `liquidation_price_usd` for each collateral asset
- ✅ Report tier in plain language with the recommended action
- ✅ Always cite `loan_term_days` when reporting probability (e.g. "12.4% over 7 days")

---

## Example Output

### Liquidation Risk Check — 0xd8dA...6045 on Ethereum (Aave V3)

```
🏦 LIQUIDATION RISK ASSESSMENT
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
Address:  0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045
Platform: AAVE (Ethereum Mainnet)
Tier:     👁 WATCH

Health Factor:           1.23
Liquidation Probability: 12.4% over 7 days
Collateral (USD):        $42,000
Debt (USD):              $30,000

Per-Asset Liquidation Prices:
  WETH  Current: $3,500  →  Liquidation: $2,730  (−21.9% drop)

Debt Positions:
  USDC  $28,000

Recommendation: Monitor closely. A ~22% drop in ETH price would trigger
liquidation. Consider adding collateral or partially repaying USDC debt.
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
```

---

## Integration Setup

### Claude Code (CLI)

```bash
claude mcp add --transport http smartcredit-aave-liquidation-monitor \
  https://mcp.smartcredit.io/web3-borrow-lend-mcp
```

### Claude Web / Claude Desktop

1. Go to **Settings → Integrations → Add integration**
2. Name: `SmartCredit Aave Liquidation Monitor`
3. URL: `https://mcp.smartcredit.io/web3-borrow-lend-mcp`

### Cursor (`mcp.json`)

```json
{
  "mcpServers": {
    "smartcredit-aave-liquidation-monitor": {
      "type": "http",
      "url": "https://mcp.smartcredit.io/web3-borrow-lend-mcp"
    }
  }
}
```

### Node.js

```javascript
import { Client } from "@modelcontextprotocol/sdk/client/index.js";
import { StreamableHTTPClientTransport } from "@modelcontextprotocol/sdk/client/streamableHttp.js";

const transport = new StreamableHTTPClientTransport(
  new URL("https://mcp.smartcredit.io/web3-borrow-lend-mcp")
);
const client = new Client({ name: "my-app", version: "1.0.0" });
await client.connect(transport);

const result = await client.callTool({
  name: "get_liquidation_risk",
  arguments: {
    user_address: "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
    hf_threshold: 1.5,
    loan_term: 7,
  },
});
console.log(result.content[0].text);
```

### Python

```python
import asyncio
from mcp import ClientSession
from mcp.client.streamable_http import streamablehttp_client

async def main():
    async with streamablehttp_client(
        url="https://mcp.smartcredit.io/web3-borrow-lend-mcp",
    ) as (read, write, _):
        async with ClientSession(read, write) as session:
            await session.initialize()
            result = await session.call_tool(
                "get_liquidation_risk",
                arguments={
                    "user_address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
                    "hf_threshold": 1.5,
                    "loan_term": 7,
                },
            )
            print(result.content[0].text)

asyncio.run(main())
```

---

## Related Subagents (Claude Code)

| Subagent | Use When |
|---|---|
| `smartcredit-liquidation-monitor` | Core risk check — tier, probability, health factor, liquidation prices |
| `smartcredit-health-advisor` | Advice on how to improve health factor and avoid liquidation |
| `smartcredit-threshold-finder` | Find exact price levels that would trigger liquidation |
| `smartcredit-stress-tester` | Stress test across 7/14/30/90-day time horizons |
| `smartcredit-portfolio-scanner` | Scan and rank multiple positions by risk |
| `smartcredit-risk-reporter` | Generate a full written risk report for documentation or sharing |

---

## Background Reading

- https://smartcredit.io/blog/aave-liquidation-risk-guide/
- https://smartcredit.io/blog/health-factor-explained/
- https://smartcredit.io/blog/defi-lending-risk-management/
- https://smartcredit.io/blog/log-normal-volatility-model-for-defi/
- https://smartcredit.io/blog/mcp-integration-guide/

---

## Requirements

- **MCP-compatible host** — Claude Code, Cursor, Claude Desktop, or any MCP client that supports Streamable HTTP transport
- **No API key** — the MCP server runs publicly at `https://mcp.smartcredit.io/web3-borrow-lend-mcp`
- **Ethereum mainnet** — only Aave V3 positions on Ethereum are supported
