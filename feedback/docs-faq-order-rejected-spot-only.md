# "My order was rejected" FAQ has no perp causes
Severity: normal
Area: docs

## Steps to reproduce
1. Open https://app.testnet.dreamdex.io/docs/support/faq/order-rejected
2. Read the causes and the "Read more" links.
3. Compare with the perp-only rejection causes listed on https://app.testnet.dreamdex.io/docs/developers/perps-testnet ("Common pitfalls") and https://app.testnet.dreamdex.io/docs/trading/perps-testnet/margin-and-liquidations ("Paper USDso", "Margin modes").

## Expected
Perps traders on testnet (Perps Arena) are the main people placing orders right now, so the rejection FAQ should cover the perp reasons too, or link to a perps section: not enough margin in the Perps account, a market outside the paper-USDso allowlist, an isolated account ordering in another market, the per-account position cap, and a stop/TP/SL without the 0.15 STT it needs.

## Actual
The page's sections are: below the minimum size; not a valid increment; Post-Only that would have filled; market order only partly filled; worse price than quoted; would have matched your own order; someone else filled ahead; if you saw an error name; still stuck. "Read more" links to Order Types and Spot Trading only. The page body has no mention of margin, leverage, paper USDso/voucher, isolated mode, the position cap or liquidation.

## Evidence
- Section headings and body text of the FAQ page, 29 Oct 2026: 0 matches for "margin", "leverage", "voucher", "Paper", "isolated", "position cap", "liquidat", "Insufficient".
- The perps developer page's "Common pitfalls" lists these as rejection causes: paper USDso blocks some markets (`limitedBy: "voucherBlocked"`), isolated accounts trade one market only (`IsolatedMarketBlocked`), per-account position cap (`MaxPositionSizeExceeded()`), stop orders need 0.15 STT.
- The perps market table marks six markets "No" under Paper USDso (1000PEPE, LIT, PUMP, VVV, ZEC, ZRO), so an Arena trader increasing a position there is rejected with no FAQ entry explaining why.

## Environment
Docs at app.testnet.dreamdex.io (29 Oct 2026), Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs check)
