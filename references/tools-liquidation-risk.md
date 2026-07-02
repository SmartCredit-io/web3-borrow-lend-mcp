# Tool Reference: `get_liquidation_risk`

## Overview

Real-time Aave V3 liquidation risk scoring for Ethereum mainnet borrower positions. Fetches live on-chain collateral and debt balances from Aave V3 contracts, then computes a z-score liquidation probability using a monthly log-normal volatility model.

Primary use case: any agent or application that needs to evaluate whether an Aave V3 borrower position is at risk of liquidation, what price levels would trigger liquidation, and how risk changes over different time horizons.

---

## Inputs

| Parameter | Type | Required | Default | Description |
|---|---|---|---|---|
| `user_address` | string | ✅ | — | Borrower wallet address (`0x…`) |
| `platform` | string | | `AAVE` | Lending protocol — only `AAVE` is currently supported |
| `hf_threshold` | float | | `1.5` | Only score positions with HF below this value; returns 204 (safe) if HF ≥ threshold |
| `loan_term` | int | | `7` | Model look-ahead window in days (1–365) |
| `sigma_window` | int | | `30` | Days of price history used for volatility estimation (7–365) |

---

## Response Codes

| Code | Meaning |
|---|---|
| `200` | Full risk profile returned (HF < hf_threshold) |
| `204` | Position is safe — HF ≥ hf_threshold; no body returned |

When you receive a 204, report: *"Position is safe — health factor is above the scoring threshold."*

---

## Output Schema (200)

```json
{
  "address": "string",               // borrower wallet address
  "platform": "string",              // lending protocol (e.g. "AAVE")
  "timestamp": "ISO-8601",           // time of assessment

  "health_factor": 0.0,             // live Aave V3 health factor
  "collateral_usd": 0.0,            // total collateral value in USD
  "debt_usd": 0.0,                  // total debt value in USD
  "collateral_ratio": 0.0,          // collateral / debt × 100 (%)
  "liquidation_ratio": 0.0,         // minimum collateral ratio before liquidation (%)

  "sigma": 0.0,                     // annualised volatility estimate from sigma_window
  "loan_term_days": 0,              // look-ahead window used in the model

  "liquidation_probability": 0.0,   // % probability of liquidation within loan_term days
  "tier": "string",                 // risk tier label (see table below)

  "collateral": [
    {
      "symbol": "string",           // token symbol (e.g. "WETH")
      "balance": 0.0,               // token balance held as collateral
      "price_usd": 0.0,             // current token price in USD
      "value_usd": 0.0,             // balance × price_usd
      "liquidation_threshold": 0.0, // Aave V3 liquidation threshold for this asset (0–1)
      "liquidation_price_usd": 0.0  // price at which this asset would trigger liquidation
    }
  ],

  "debt": [
    {
      "symbol": "string",           // token symbol (e.g. "USDC")
      "balance": 0.0,               // outstanding debt balance
      "price_usd": 0.0,             // current token price in USD
      "value_usd": 0.0              // balance × price_usd
    }
  ]
}
```

---

## Risk Tier Table

| Tier | Liquidation Probability | Recommended Action |
|---|---|---|
| `Safe` | 0–5% | No action needed |
| `Watch` | 5–15% | Monitor closely |
| `Warning` | 15–35% | Consider reducing exposure |
| `Danger` | 35–70% | Reduce position urgently |
| `Critical` | 70–100% | Immediate action required |
| `Liquidatable` | HF < 1.0 | Position is liquidatable now |

---

## Collateral Array Fields

| Field | Type | Description |
|---|---|---|
| `symbol` | string | Token ticker (e.g. `WETH`, `WBTC`) |
| `balance` | float | Token units supplied as collateral |
| `price_usd` | float | Current market price per token |
| `value_usd` | float | Current USD value of collateral (`balance × price_usd`) |
| `liquidation_threshold` | float | Aave V3 liquidation threshold for this asset (0–1). E.g. `0.825` means the position can be liquidated when collateral value drops to 82.5% of debt |
| `liquidation_price_usd` | float | The price of this asset at which the position becomes liquidatable |

To compute **% drop to liquidation** for an asset:
```
% drop = ((price_usd - liquidation_price_usd) / price_usd) × 100
```

---

## Debt Array Fields

| Field | Type | Description |
|---|---|---|
| `symbol` | string | Token ticker (e.g. `USDC`, `DAI`) |
| `balance` | float | Outstanding debt units |
| `price_usd` | float | Current market price per token |
| `value_usd` | float | Current USD value of debt |

---

## Error Cases

| Code | Cause |
|---|---|
| `400 Bad Request` | Malformed `user_address` or unsupported `platform` |
| `404 Not Found` | No active Aave V3 position found for the address |
| `500 Internal Server Error` | Temporary downstream failure (RPC or price feed) |

---

## Key Model Notes

- **Log-normal volatility model:** The tool uses a monthly log-normal distribution fitted to `sigma_window` days of price history to compute the z-score probability that the position's collateral value drops below the liquidation threshold within `loan_term` days.
- **`sigma`** in the response is the annualised volatility estimate derived from that window. Higher sigma → higher liquidation probability for the same health factor.
- **`hf_threshold` behaviour:** If the live health factor is ≥ `hf_threshold`, the tool returns 204 immediately without running the model. This is a performance optimisation — well-collateralised positions are treated as out-of-scope.
- **`health_factor < 1.0`** means the position is already undercollateralised and can be liquidated by any liquidator right now.

---

## Usage in Agents

`get_liquidation_risk` is used by all SmartCredit subagents:

| Agent | How it uses this tool |
|---|---|
| `smartcredit-liquidation-monitor` | Single call with defaults; formats the result as a risk report |
| `smartcredit-health-advisor` | Single call; uses collateral/debt breakdown to generate improvement advice |
| `smartcredit-threshold-finder` | Single call; extracts `liquidation_price_usd` from `collateral[]` |
| `smartcredit-stress-tester` | Four calls with `loan_term` = 7, 14, 30, 90; compares results |
| `smartcredit-portfolio-scanner` | One call per address in a list; ranks results by `liquidation_probability` |
| `smartcredit-risk-reporter` | Single call with optional model params; writes a structured report |

---

## Example Prompts

- *"What is the liquidation risk for 0xd8dA6BF...?"*
- *"Check if this borrower is close to liquidation on Aave."*
- *"What is the health factor for this wallet?"*
- *"At what price does WETH trigger liquidation for this position?"*
- *"Evaluate this borrower's collateral safety over 30 days."*
- *"Is 0xABC... at risk of being liquidated on Aave V3?"*
