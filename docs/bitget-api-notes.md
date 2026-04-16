# BitGet API Notes

## Demo trading — hard-won findings

BitGet's simulated trading environment has several non-obvious requirements that differ from their docs:

| What | Wrong | Correct |
|---|---|---|
| Demo header value | `papTrading: "true"` | `papTrading: "1"` |
| Futures order endpoint | `/api/v2/mix/order/placeOrder` | `/api/v2/mix/order/place-order` |
| Futures `size` unit | number of contracts | BTC amount (4 dp, min `0.0001`) |
| Futures margin mode | `"isolated"` | `"crossed"` (demo account default) |
| Spot endpoint in demo | works | **not supported** — 404s always |

## Demo API key setup

Keys must be generated from within BitGet's Simulated Trading mode:
BitGet → top-right toggle → Simulated Trading → API Management → Create key.

- Demo keys only work **with** `papTrading: "1"`
- Live keys only work **without** it
- Demo keys have no IP whitelist setting — permissions only

## Demo account defaults

- Comes pre-funded with $100 USDT in the futures wallet
- Default margin mode: `crossed`
- At ~$75k BTC, minimum order (0.0001 BTC) = ~$7.50 notional
- Set leverage ≥ 10x if the balance can't cover the margin requirement

## Futures order requirements

`tradeSide: "open"` is required on entry orders or the request is rejected.

Full working order body:
```json
{
  "symbol": "BTCUSDT",
  "productType": "USDT-FUTURES",
  "marginMode": "crossed",
  "marginCoin": "USDT",
  "side": "buy",
  "tradeSide": "open",
  "orderType": "market",
  "size": "0.0001"
}
```

## Switching to live trading

1. Generate API keys from your **live** BitGet account (not simulated mode)
2. Update `.env` with the new credentials
3. Set `BITGET_DEMO=false` (removes the `papTrading` header)
4. Set Railway: `railway variable set BITGET_DEMO=false`
