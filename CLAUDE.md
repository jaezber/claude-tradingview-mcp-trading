# Claude Trading Bot — Project Handover

## What this is

An automated trading bot that checks TradingView indicator values against a strategy defined in `rules.json`, then executes trades on BitGet. Runs on a cron schedule in Railway (cloud).

## Setup status — COMPLETE

Everything has been set up. Do not re-run onboarding. The bot is live and deployed.

## Key details

| Setting | Value |
|---|---|
| Exchange | BitGet (USDT-futures, demo account) |
| Symbol | BTCUSDT, SOLUSDT, ETHUSDT, DOGEUSDT |
| Timeframe | 4H |
| Portfolio value | $1,000 |
| Max trade size | $50 |
| Max trades/day | 3 |
| Mode | **DEMO TRADING** (real API calls, simulated money) |
| Cloud schedule | Every 4 hours (`0 */4 * * *`) |
| GitHub | https://github.com/jaezber/claude-tradingview-mcp-trading |

## Files

| File | Purpose |
|---|---|
| `bot.js` | Main bot — run with `node bot.js` |
| `rules.json` | Strategy conditions (VWAP + RSI(3) + EMA(8) scalping) |
| `.env` | API credentials and trading limits (never commit this) |
| `railway.json` | Railway deployment config (cron schedule lives here) |
| `trades.csv` | Full trade log — open in Google Sheets / Excel |
| `safety-check-log.json` | Audit trail of every bot decision |

## Strategy (rules.json)

VWAP + RSI(3) + EMA(8) scalping on the 1-minute chart.

**Long entry — ALL must be true:**
- Price above VWAP (buyers in control)
- Price above EMA(8) (uptrend confirmed)
- RSI(3) below 30 (snap-back setup)

**Short entry — ALL must be true:**
- Price below VWAP (sellers in control)
- Price below EMA(8) (downtrend confirmed)
- RSI(3) above 70 (overbought in downtrend)

**Risk rules:**
- Max 1% portfolio risk per trade
- Hard stop: 0.3% from entry
- No trade if price >1.5% from VWAP

## Railway deployment

- Project: `alluring-fascination`
- Service: `alluring-fascination`
- URL: https://railway.com/project/3c32f1ef-b245-4de1-9af8-e76d0b3724b3
- Env vars set: `PAPER_TRADING=false`, `BITGET_DEMO=true`, `TRADE_MODE=futures`

**To go live (real money):**
```bash
railway variable set BITGET_DEMO=false
# Also swap .env credentials to your live BitGet API keys
```

**To check logs:**
```bash
railway logs
```

**To redeploy after code changes:**
```bash
railway up
```

## BitGet demo trading — hard-won API notes

BitGet's simulated trading environment has several non-obvious requirements that differ from their docs:

| What | Wrong | Correct |
|---|---|---|
| Demo header value | `papTrading: "true"` | `papTrading: "1"` |
| Futures order endpoint | `/api/v2/mix/order/placeOrder` | `/api/v2/mix/order/place-order` |
| Futures `size` unit | number of contracts | BTC amount (4 dp, min `0.0001`) |
| Futures margin mode | `"isolated"` | `"crossed"` (demo account default) |
| Spot endpoint in demo | works | **not supported** — 404s always |

**Demo API keys** must be generated from within BitGet's Simulated Trading mode:
BitGet → top-right toggle → Simulated Trading → API Management → Create key.
Demo keys only work with `papTrading: "1"`. Live keys only work without it.

**Demo account comes with $100 USDT** in the futures wallet. At $75k BTC, the minimum
order (0.0001 BTC) costs ~$7.50 notional. Set leverage ≥ 10x if needed.

**`tradeSide: "open"`** is required on futures entry orders or the request is rejected.

## TradingView MCP

The TradingView MCP (68 tools for reading live chart data) is installed at:
```
/Users/jespervestergaard/tradingview-mcp-jackson
```

Health check tool: `tv_health_check` — returns `cdp_connected: true` when TradingView Desktop is running with CDP enabled.

## Running the bot manually

```bash
node bot.js
```

This runs one full cycle: fetches live indicator values, checks all strategy conditions, logs the decision, and executes a trade if all conditions pass (paper mode by default).

```bash
node bot.js --tax-summary
```

Prints total trades, volume, and estimated fees to date.

## Tax records

All trades (including blocked ones with failure reasons) are logged to `trades.csv`. Import directly into Google Sheets, Excel, or accounting software at tax time.
