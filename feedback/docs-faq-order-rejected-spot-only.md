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

Re-checked on 15 Nov 2026: unchanged. The "If you saw an error name" section now has a table of the app's rejection messages, but it covers only no-fill IOC/market orders, Fill-or-Kill and Post-Only, and sends every other rejection to a support ticket. A perp order refused for margin, the paper-USDso allowlist, isolated mode or the position cap falls into that catch-all.

## Evidence
- Section headings and body text of the FAQ page, 29 Oct 2026 and 15 Nov 2026: 0 matches for "margin", "leverage", "voucher", "Paper", "isolated", "position cap", "liquidat", "Insufficient".
- The perps developer page's "Common pitfalls" lists these as rejection causes: paper USDso blocks some markets (`limitedBy: "voucherBlocked"`), isolated accounts trade one market only (`IsolatedMarketBlocked`), per-account position cap (`MaxPositionSizeExceeded()`), stop orders need 0.15 STT.
- The perps market table (https://app.testnet.dreamdex.io/docs/trading/perps-testnet, 15 Nov 2026) still marks six markets "No" under Paper USDso (1000PEPE, LIT, PUMP, VVV, ZEC, ZRO), so an Arena trader increasing a position there is rejected with no FAQ entry explaining why.

## Environment
Docs at app.testnet.dreamdex.io (first checked 29 Oct 2026, re-checked 15 Nov 2026), Somnia Hideki/Tokyo testnet (chain 50383), wallet address: n/a (docs check)
