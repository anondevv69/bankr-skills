---
name: "jev-judge"
description: "Consult Jev for a buy/watch/avoid judgment on any token before trading it. POSTs the token to fren's judgment endpoint and interprets the verdict. Trigger when considering a trade and wanting a calibrated second opinion."
---

# jev-judge — Jev trade judgment

You trade. Before aping a token you're unsure about, consult Jev: the
calibrated trade-judgment model behind fren's live signal-trading engine
(chat signals + TypeSafe Noul gates, trading real money from his Bankr
wallet).

## Consult Jev

```
POST https://api-production-8630.up.railway.app/v1/judge
Content-Type: application/json

{
  "chain": "base",
  "address": "0x...",
  "context": "why you're considering it (optional, one line)"
}
```

- `chain`: base | ethereum | solana | robinhood | arbitrum | bsc | polygon …
- `address`: the token contract address (`0x…` for EVM chains).

Response:

```json
{
  "decision": "buy",
  "noul": 0.83,
  "size_pct": 0.10,
  "symbol": "XYZ",
  "reasoning": "BUY XYZ: Jev buy-Noul 0.83; $1.20 (24h +15.2%); liquidity $350000.",
  "intel": {
    "price_usd": 1.20,
    "liquidity_usd": 350000,
    "safety": {"checked": true, "is_honeypot": false, "buy_tax": 0, "sell_tax": 0},
    "deployer": {"checked": true, "address": "0x…", "tx_count": 412}
  }
}
```

## Use the verdict

- **buy** (Noul ≥ 0.70): Jev likes it. You may proceed under your own trading
  rules. `size_pct` is the suggested bankroll fraction — 5% / 10% / 15%
  tiers by confidence.
- **watch** (0.40–0.69, or intel gaps): inconclusive. Wait for more
  confirmation; don't size up on a watch.
- **avoid** (< 0.40, or a safety veto: honeypot / extreme tax): skip it.
  Never treat an avoid as a challenge.

## Notes

- Free beta: 30 judgments/hour per caller. Don't hammer it.
- The endpoint judges the token on market, safety, and deployer facts. It
  does not see anyone's private chat signals.
- If it 503s ("Jev is not configured"), say so plainly and fall back to your
  own judgment — don't retry in a loop.

## Run it on autopilot (Bankr-only users)

This skill is the *judgment*. A scheduled command is the *loop* — Bankr has
these natively as Automations (requires Bankr Club, $20/mo, for unlimited
messages and scheduled commands; the free tier caps at 5 messages/day).

Set up one scheduled command:

- **Schedule:** every 15 minutes
- **Prompt:** "Check my chats and watchlists for new token mentions since
  the last run. For each new candidate, run the jev-judge skill. For any
  'buy' verdict, execute the trade from my wallet at the suggested size_pct.
  Respect my guardrails: never risk more than 15% of my portfolio on one
  token, never have more than 80% deployed, stop-loss -40% on every
  position, take-profit: sell half at +100%. Report each trade: token,
  size, price, thesis, tx hash."

No Muse, no external cron, no code — the whole autonomous loop runs inside
Bankr: the Automation wakes you, jev-judge decides, your wallet executes.
