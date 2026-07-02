# SmartCredit Aave Liquidation Monitor MCP

## Project Overview

This repository contains the **SmartCredit Aave Liquidation Monitor MCP** — a real-time liquidation risk intelligence layer for Aave V3 borrower positions on Ethereum mainnet.

- **MCP Endpoint:** `https://mcp.smartcredit.io/web3-borrow-lend-mcp`
- **Transport:** Streamable HTTP (MCP spec 2025-03-26)
- **Access:** Public — no authentication required
- **Network:** Ethereum mainnet (Aave V3 only)
- **GitHub:** `https://github.com/SmartCredit-io/web3-borrow-lend-mcp`
- **Website:** https://smartcredit.io

---

## MCP Tools (2 total)

| Tool | Purpose |
|---|---|
| `get_liquidation_risk` | Evaluate liquidation probability for an Aave V3 borrower position using a log-normal volatility model |
| `check_api_health` | Ping the server to confirm it is reachable |

---

## Repository Structure

```
web3-borrow-lend-mcp/
├── .claude/
│   └── agents/              # 6 Claude Code subagents
├── agents/
│   └── openai.yaml          # Codex/OpenAI metadata
├── assets/
│   └── README.md            # Logo and brand assets placeholder
├── references/              # Deep tool documentation
│   ├── tools-liquidation-risk.md
│   └── tools-health.md
├── AGENTS.md                # Machine-readable subagent index
├── CLAUDE.md                # This file
├── SKILL.md                 # Skill definition (Claude Code, Cursor, OpenClaw)
├── llms.txt                 # LLM context file
├── glama.json               # Glama MCP registry metadata
├── server.json              # MCP server registry metadata
└── README.md                # Full documentation and integration guides
```

---

## Subagents

6 specialist subagents in `.claude/agents/`. Use the right one for the task:

| Agent | Model | Tools Used | Use For |
|---|---|---|---|
| `smartcredit-liquidation-monitor` | Haiku | `get_liquidation_risk` | Core liquidation risk check for a single position |
| `smartcredit-health-advisor` | Sonnet | `get_liquidation_risk` | Actionable advice to improve health factor |
| `smartcredit-threshold-finder` | Haiku | `get_liquidation_risk` | Per-asset liquidation price thresholds |
| `smartcredit-stress-tester` | Sonnet | `get_liquidation_risk` | Multi-horizon risk matrix (7/14/30/90 days) |
| `smartcredit-portfolio-scanner` | Sonnet | `get_liquidation_risk` | Batch scan multiple positions, ranked by risk |
| `smartcredit-risk-reporter` | Sonnet | `get_liquidation_risk` | Full written risk report suitable for documentation |

---

## Risk Tiers

| Tier | Liquidation Probability | Action |
|---|---|---|
| Safe | 0–5% | No action needed |
| Watch | 5–15% | Monitor closely |
| Warning | 15–35% | Consider reducing exposure |
| Danger | 35–70% | Reduce position urgently |
| Critical | 70–100% | Immediate action required |
| Liquidatable | HF < 1.0 | Position is liquidatable now |

---

## Conventions

- **Haiku** for fast, single-address, deterministic lookups (monitor, threshold-finder)
- **Sonnet** for agents requiring reasoning across multiple calls or generating written output (health-advisor, stress-tester, portfolio-scanner, risk-reporter)
- No API key required — the MCP endpoint is public
- All positions are on Ethereum mainnet (Aave V3); the `platform` parameter currently only accepts `AAVE`
- A **204 response** from `get_liquidation_risk` means the position's HF is above the `hf_threshold` — treat as safe
- `health_factor < 1.0` means the position is already liquidatable

---

## Tool Reference Docs (this repo)

- `references/tools-liquidation-risk.md` — `get_liquidation_risk` full schema, parameters, output fields, risk tiers
- `references/tools-health.md` — `check_api_health` usage

---

## Setup (Claude Code)

```bash
claude mcp add --transport http smartcredit-aave-liquidation-monitor \
  https://mcp.smartcredit.io/web3-borrow-lend-mcp
```

```bash
git clone https://github.com/SmartCredit-io/web3-borrow-lend-mcp.git
cp -r web3-borrow-lend-mcp/.claude/agents/ your-project/.claude/agents/
```
