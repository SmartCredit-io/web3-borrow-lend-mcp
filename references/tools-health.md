# Tool Reference: `check_api_health`

## Overview

A lightweight ping endpoint to verify that the SmartCredit MCP server is reachable and operational. Use this before running a batch of liquidation risk checks or when troubleshooting connectivity issues.

---

## Inputs

None. This tool takes no parameters.

---

## Output

Returns a plain string:

| Response | Meaning |
|---|---|
| `"ok"` | Server is reachable and operational |
| Error string | Server is unreachable or returning an error; include the message in your response |

---

## When to Use

- Before a batch scan to confirm the server is up
- When `get_liquidation_risk` returns unexpected errors and you want to rule out connectivity issues
- As a startup check in automated pipelines that depend on the MCP server

---

## Example

**Prompt:** *"Is the SmartCredit MCP server online?"*

**Response:**
```
Server status: ok
The SmartCredit Aave Liquidation Monitor is reachable and operational.
```

---

## Notes

- This tool does not authenticate and does not require a wallet address or any other input.
- A successful response (`"ok"`) does not guarantee that downstream data sources (Aave V3 RPC, CoinGecko, Chainlink) are healthy — it only confirms the MCP server itself is responding.
- If you need to confirm full data pipeline health, make a test `get_liquidation_risk` call with a known active address.
