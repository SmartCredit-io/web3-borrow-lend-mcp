# Aave Liquidation Monitor MCP Server

## Overview

SmartCredit operates a hosted MCP server delivering real-time liquidation risk intelligence for **Aave V3** borrower positions on Ethereum mainnet. The platform uses a log-normal volatility model to compute liquidation probability, per-asset price thresholds, and tiered risk labels — queryable from any MCP-compatible AI client in a single tool call.

**Key Details:**

- **Server URL:** `https://mcp.smartcredit.io/web3-borrow-lend-mcp`
- **Transport:** Streamable HTTP (MCP spec 2025-03-26)
- **Status:** Hosted, always-on
- **Access:** Public, no authentication required
- **Network:** Ethereum mainnet (Aave V3)
- **Data sources:** Aave V3 contracts (live RPC), CoinGecko price feed, Chainlink fallback

---

## Tools

### 1. `get_liquidation_risk`

Evaluate the liquidation risk for a single Aave V3 borrower position.

Fetches live on-chain collateral and debt balances, computes a z-score liquidation probability using a monthly log-normal volatility model, and returns a full risk profile including per-asset liquidation price thresholds and stress scenarios.

**Parameters**

| Name | Type | Required | Default | Description |
|---|---|---|---|---|
| `user_address` | string | ✅ | — | Borrower wallet address (`0x…`) |
| `platform` | string | | `AAVE` | Lending protocol. Only `AAVE` supported. |
| `hf_threshold` | float | | `1.5` | Only score positions with HF < this value. Returns "safe" otherwise. |
| `loan_term` | int | | `7` | Model look-ahead window in days (1–365). |
| `sigma_window` | int | | `30` | Days of price history for volatility estimation (7–365). |

**Returns** a JSON risk profile (200) or a "position is safe" message (204).

---

### 2. `check_api_health`

Ping the server to confirm it is reachable. No parameters. Returns `"ok"` or an error description.

---

## Risk Tiers

| Tier | Liquidation Probability | Action |
|---|---|---|
| **Safe** | 0–5% | No action needed |
| **Watch** | 5–15% | Monitor closely |
| **Warning** | 15–35% | Consider reducing exposure |
| **Danger** | 35–70% | Reduce position urgently |
| **Critical** | 70–100% | Immediate action required |
| **Liquidatable** | HF < 1.0 | Position is liquidatable now |

---

## Integration

### Claude Code CLI

```bash
claude mcp add --transport http aave-liquidation-monitor \
  https://mcp.smartcredit.io/web3-borrow-lend-mcp
```

### Claude Desktop

Add to `claude_desktop_config.json` (macOS: `~/Library/Application Support/Claude/`, Windows: `%APPDATA%\Claude\`):

```json
{
  "mcpServers": {
    "aave-liquidation-monitor": {
      "type": "http",
      "url": "https://mcp.smartcredit.io/web3-borrow-lend-mcp"
    }
  }
}
```

### Cursor / Windsurf

Add to `.cursor/mcp.json` or `~/.config/windsurf/mcp.json`:

```json
{
  "mcpServers": {
    "aave-liquidation-monitor": {
      "type": "http",
      "url": "https://mcp.smartcredit.io/web3-borrow-lend-mcp"
    }
  }
}
```

### Node.js (MCP SDK)

```js
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

### Python (MCP SDK)

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

## Example Response

```json
{
  "address": "0xd8dA6BF26964aF9D7eEd9e03E53415D37aA96045",
  "platform": "AAVE",
  "timestamp": "2026-07-02T09:15:00Z",
  "health_factor": 1.23,
  "collateral_usd": 42000.00,
  "debt_usd": 30000.00,
  "collateral_ratio": 140.0,
  "liquidation_ratio": 76.9,
  "sigma": 0.38,
  "loan_term_days": 7,
  "liquidation_probability": 12.4,
  "tier": "Watch",
  "collateral": [
    {
      "symbol": "WETH",
      "balance": 10.0,
      "price_usd": 3500.00,
      "value_usd": 35000.00,
      "liquidation_threshold": 0.825,
      "liquidation_price_usd": 2730.00
    }
  ],
  "debt": [
    {
      "symbol": "USDC",
      "balance": 28000.00,
      "price_usd": 1.00,
      "value_usd": 28000.00
    }
  ]
}
```

---

## Access

The server is publicly accessible — no API key or authentication required.

**License:** MIT (client SDK examples) · Proprietary backend
